---
layout: draft
title:  "boris’ practical tips (life, machine learning, and everything)"
date:   2018-04-21
use_math: true
---

# boris’ practical tips (life, machine learning, and everything)
after a 3 year tenure, last week was my last week at orbital insight.  before i left, someone from my team asked me to write down some practical tips/advice for training deep learning models, and since I personally hate unsolicited advice, i decided to extend the scope to include a lot of unsolicited advice about things far beyond deep learning.

## life / career
- avoid the “interesting trap”: working on something that is interesting is rewarding in the near term, but working on something *useful* (either to your team-mates or to customers) is rewarding in the long term.  the two are not mutually exclusive, but people with an intellectual leaning tend to place an unproportional weight on “interesting”.  stop yourself periodically and ask whether you’re balancing the two aspects well.
- don’t play violin at 2am.  especially if you have roommates.



## engineering
- write unit tests.
  - if you find yourself checking your code by trying toy examples in a notebook or in the ipython shell, you should probably write a unit test.
  - if you discover a bug, you should probably write a unit test (write the test first to repro bug, make sure it fails, then fix the bug and make sure test passes).
- naming things is hard, but also important.  name your variables well.  some examples particularly relevant to machine learning:
  - if your variables are meant to represent physical quantities, include units in the variable name, e.g. `length_mtr` or `length_px` (in CV, we often use pixels as a unit of distance) versus just `length`, `timeout_ms` or `timeout_sec` versus just `timeout`, `angle_rad` or `angle_deg` versus just `angle`, etc.
  - `x` and `y` conventions get very confusing… e.g. in numpy, accessing an element in a 2D matrix is done via `(row, col)`, whereas specifying a location in a 2D space is usually done via `(x, y)`… same goes for specifying the size of a matrix — in numpy it’s `(num_rows, num_cols)`, versus `(width, height)` for more physical quantities.  so, if you have a tuple with coordinates, prefer names like `coords_xy` or `coords_rc` over just `coords`, and for shapes/size, prefer `*_shape` to mean rows/cols, and `*_size` to mean (width, height).
  - angles are the worst.  clockwise versus counter-clockwise… relative to positive x-axis? or positive y?  this isn’t really a tip, just a complaint…
- occam’s razor: don’t do anything complicated until you’ve tried simple things and ensured they don’t work.  simple things are more likely to work well.  complicated things are hard to maintain, debug, etc.  prefer simple things over complicated things even if the simple thing is a little bit worse (unless performance is absolutely critical).  this is related the the “interesting trap” from above -- interesting things tend to be complicated, so people tend to be pulled towards them.
- (when working in a larger engineering org) if you see something that looks wrong, never say “i don’t know what’s going on here… meh, not my problem”.   either dig in yourself to figure out the problem, or escalate the problem to someone else.


## machine learning / computer vision / deep learning
- the **most** important thing to getting good results is how you frame the problem.  for example, if you want to categorize pictures of birds, should you set that up as an image classification task?  or is it better to get ground truth for parts of each bird (wings, head, tail, etc) and have the model take that into account during training/inference?  which one will be easier for the model to learn? which one will be easier for human annotators to collect accurate ground truth?  how should you sample the images you get annotated? and so on…
- the **second** most important thing to getting good results is the dataset.  is it large enough? is it accurate enough?  is it representative of the data the model will be used on?
- the **least** ******important thing to getting good results is the model/algorithm (the least out of this short list — it’s still important!)  unfortunately, it’s also usually the *most* interesting thing to work on, compared to the two things above.  see “interesting trap” above.
- avoid “training-serving skew” -- this is *the* most common source of problems in ML-based products.  as mentioned above, think carefully about how the data is sampled.  make sure that you do the exact same pre-processing during training and production/serving (ideally, use the same exact piece of code).
- avoid overfitting.  always be mindful of this.  you always want to get good results, so you always have an incentive to (unintentionally) overfit.  
- class imbalance is a common problem.  here are some tricks we usually use to deal with it:
  - tweak class weights in the loss function (e.g. see the `class_weights` arg into the `fit` method in [keras](https://keras.io/models/model/#fit))
  - how you sample your batches is very important -- e.g. you may want to sample “hard” examples of a common category more frequently (as in “hard negative mining”), and/or ensure that each batch contains enough examples of the rare categories/labels
- run hyper-parameter searches with a smaller dataset, and then use the “complete” large dataset to train on the best hyper-parameters, otherwise you you won’t be able to iterate on the model as quickly.
- make sure the model has the right receptive field size (more relevant for semantic segmentation type tasks).  get intuition by cropping out patches of a given size from the dataset and see if you can classify the center pixel, or if more context is needed.
- read [Bengio’s paper](https://arxiv.org/abs/1206.5533), which has a wealth of tips, as well as the [Stanford CS231N course notes](http://cs231n.github.io)
