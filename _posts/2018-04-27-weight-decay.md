---
layout: draft
title:   weight decay vs L2 regularization"
date:   2018-04-27
use_math: true
---

one popular way of adding regularization to deep learning models is to include a weight decay term in the updates.  this is the same thing as adding an $L_2$ regularization term to the loss… or is it?

## a brief story about a bug

machine learning bugs are notoriously difficult to track down — rather than getting an exception, or a completely wrong output, the model might perform a little bit worse than it should.  i had one such experience when moving some code over from caffe to keras a few months ago.  i combed the code to make sure all hyperparameters were exactly the same, and yet when i would train the model on the exact same dataset, the keras model would always perform a bit worse.  the difference ended up being due to the subtle difference between “weight decay” and “$L_2$ regularization”.

## the subtle difference

at the end of the day the difference is just in terminology (and i’m guessing the literature is inconsistent), but one that can have an important practical difference.  weight decay is usually defined as a term that’s added directly to the update rule.  e.g., in the seminal [AlexNet paper](https://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks.pdf):

![png](/assets/posts/weight_decay/weight_decay_alexnet.png)


where $L$ is your typical loss function (e.g. cross entropy).  in the above, the authors use a weight decay hyperparameter of 0.0005.

on the other hand, $L_2$ regularization is added to the loss, i.e. the optimization becomes:

$$\text{argmin}_w L_{\text total} = \text{argmin}_w \Big(L + \lambda||w||_2^2\Big)$$

here, $$L_{\text total}$$ is what we’ll call “total loss” which combines the loss $$L$$ with the regularization.  when you take a derivative of the above, the update becomes (ignoring momentum to keep this simple):

$$w_{i+1} = w_i - 2 \lambda w_i - \Big<\frac{\delta L}{\delta w}|_{w_i}\Big>$$

the key difference is the pesky factor of 2!  so, if you had your weight decay set to 0.0005 as in the AlexNet paper and you move to a deep learning framework that implements $L_2$ regularization instead, you should set that $$\lambda$$ hyperparameter to 0.0005/2.0 to get the same behavior.  this is what ended up causing the difference when moving from caffe (which implements weight decay) to keras (which implements $L_2$ regularization) — i set this hyperparameter to the same value in both.  note: some frameworks may define the $L_2$ term with a 0.5 in front so that it cancels out the factor of 2 in the gradient, but that is [not the case with keras](https://github.com/keras-team/keras/blob/master/keras/regularizers.py#L42).

## fancy solvers

the above issue alone can cause plenty of headache, but the story does not end there.  consider “fancy” solvers like adadelta and adam, which incorporate the history of gradients in some way.  in frameworks that implement $L_2$ regularization, the gradients the solver is using are for “total loss” $L_{\text total}$ above, which has $L_2$ regularization baked into it (this fact is abstracted away from the solver — it doesn’t “know” anything about whether a regularization term was included in the loss or not).  in frameworks that implement weight decay, the solver is considering only the gradient for $L$.  therefore, the exact manner that a deep learning framework implements weight decay/regularization will actually affect what these solvers will do.  unfortunately, this is difference is much trickier to remove compared to the “divide this hyperparameter by 2” solution above…

is one implementation better than the other?  a [recent paper by loshchilov et al.](https://arxiv.org/abs/1711.05101) (shown to me by my co-worker [Adam](https://twitter.com/adamwkraft), no relation to the solver) argues that the weight decay approach is more appropriate when using fancy solvers like Adam.

## tensorflow layers API

this last bit is a quick aside: i was flipping through the [official tutorial for the tensorflow layers API (r1.7 as of this writing)](https://www.tensorflow.org/tutorials/layers) (which looks very similar to keras), and was wondering how to configure regularization.  it turns out, similar to keras, when you create layers (either via the class or the function), you can pass in a regularizer object.  there’s a big gotcha though — if you try to extend the tutorial i linked to above to include regularization, it won’t work!  in the totural, the loss tensor that’s passed into the estimator is defined as:

```python
loss = tf.losses.sparse_softmax_cross_entropy(labels=labels, logits=logits)
```

i started digging around to see if there’s some magic happening behind the scenes to pick up the regularizers you've passed into the layers and add them to the loss inside the estimator — keras does this sort of [magic](https://github.com/keras-team/keras/blob/master/keras/engine/training.py#L845) for you, but the estimator code does not.  passing the regularizers into the layers simply results in those regularization tensors into the `REGULARIZATION_LOSSES` collection, but it’s up to the caller to pick these up, add them to the main loss and pass that combination into the estimator.  alternatively, if you register the main loss into the `LOSSES` collection, you can then call [this function](https://www.tensorflow.org/api_docs/python/tf/losses/get_total_loss) to get the total loss.  all in all, this seems like a leaky abstraction to me… the layers API makes it look like it’s very easy to set regularizers per layer, but the caller needs to understand how these things work under the hood to use this feature properly.
