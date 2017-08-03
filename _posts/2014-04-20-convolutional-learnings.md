---
title:  "convolutional learnings: things i learned by implementing convolutional neural nets"
date:   2014-04-20
---

deep learning has taken the machine learning world by storm, achieving state of the art results in a number of applications.  whether neural nets are here to stay or will be replaced be the next hot thing two years from now, one thing is certain: they are now a critical component in any machine learning expert’s toolbox (along with svm, random forests, etc).

aside from the basics, i didn’t know much about neural nets in depth so i decided to teach myself.  my philosophy is that the best way to learn something is by doing.  i did start off by going through geoff hinton’s coursera class, and that was a great starting point, but ultimately i decided to implement my own simple neural net (or convolutional neural net to be exact, since i do have an inclination towards vision).  since implementation details rarely make into academic literature (see this clip), i decided to take notes about the practical challenges i ran into and share them with the world.  i’ve also posted [my code on github](https://github.com/bbabenko/simple_convnet).  note that there are many neural net packages that are significantly more feature complete and efficient than this one – i am posting it purely for pedagogical reasons, since most of the other packages can be quite intimidating for someone who wants to dig in and understand how exactly they were implemented.

before i get into my key take aways, here are a few resources i found helpful:

* as mentioned earlier, [hinton’s coursera class](https://www.coursera.org/course/neuralnets) is a fantastic starting point

* yoshua bengio has an excellent technical report called [“Practical recommendations for gradient-based training of deep architectures”](http://arxiv.org/abs/1206.5533), which i found to be full of invaluable practical advice, much of which i’ll reiterate below

* the UFLDL tutorial contains some notes and homework problems, along with some starter [matlab code](http://ufldl.stanford.edu/tutorial/index.php/UFLDL_Tutorial)

* andrej karpathy has implemented [neural nets in javascript](http://cs.stanford.edu/people/karpathy/convnetjs/) that you can run in your browser and modify learning parameters on the fly

there are also a few “more serious” implementations that are worth looking through:

* cuda convnet http://cs.stanford.edu/people/karpathy/convnetjs/

* decaf and caffe http://caffe.berkeleyvision.org/

and now on to the learnings:

### numerical stability

i very quickly ran into bugs that involved numerical stability, namely the exp and log functions.  i found it useful to implement wrappers around those methods that checked the input, clipped it to an acceptable range, and issued a warning if the clipping was necessary.

### mo’ layers less problems

in an object oriented language, neural nets are usually implemented by defining an abstract layer class, and many different children classes (e.g. pooling layer, convolutional layer, etc).  i found it much easier to get the implementations right by keeping each layer as simple as possible.  for example, i separated non-linearity and bias terms into their own layers, rather than baking that into the other layers.  not only did this make it easier to reuse code (since, e.g.,  many different types of layers can use a bias), it made the calculation of gradients simpler as well by breaking them up into many small components (if you think of neural net as a series of nested functions, then back-prop boils down to applying the chain rule over and over again).  admittedly, there are some downside to this: the definition of your neural net becomes pretty verbose and it’s easier to forget to, e.g., include a bias layer where there should be one; you also sacrifice some efficiency when you calculate the results of propagating through the network.

### conv layer is the trickiest

i’m not sure how to make this point constructive… the backprop in the convolutional layer was tricky and you should brace yourself if trying to implement it yourself.  paper and pencil come in handy :)

### gradient checking is a must (unit tests)

this goes without saying, but unit tests are critical to ensuring your code is correct (not only after you first implement it, but also as you continue building on top of it).  i wish i had learned better habits in grad school (i tested my code, but it was a much more manual process), but it really wasn’t until working at dropbox that i really picked it up.  for neural nets there is a very systematic check that you can perform to verify the correctness of your derivatives – you simply calculate an approximate gradient (which is much slower than back-prop, hence it doesn’t get used in practice for gradient descent) and compare it to the one you get from back-prop.  doing this helped me catch dozens of bugs during development.

### re-using blocks of allocated memory

doing forward and backward propagation for a fixed number of data points requires a fixed and fairly large amount of memory.  allocating this memory every time you perform a gradient descent step is wasteful – it makes sense to allocate memory once and re-use it for each step.  i actually didn’t do this in my implementation since i realized this too late (and since it would have arguably increased the complexity of the implementation).  if you peruse the code for ConvNetJS and Caffe you can see “volume” or “blob” data structures used for this purpose.

### picking params is very difficult, even to get training error down! [1]

![](/assets/posts/basic_cnn/hyperparameters.gif)

one great suggestion in the bengio paper i linked above is to do what he calls “controlled overfitting”.  this means you take a small subset of data, and try to get the training error rate down to 0 (or close to that).  if your net isn’t able to do that, either you have a bug or your parameters are set wrong.

---

in the end, my simple implementation is able to achieve reasonable test error rates on mnist (<10%).  i also tried it on the harder cifar10 dataset, and there i was only able to get to around 60% accuracy.  that’s better than random, but far from the other, more solid implementations of conv nets.  there are a few possible reasons for this: 1) wrong parameter settings, 2) because my implementation is not very efficient, i wasn’t able to juice up the number of parameters without running out of memory, and 3) for the above reasons, training a hundred epochs would take forever and i don’t have that kind of patience :-P, and 4) i skipped a lot of bells and whistles like dropout and jittering images during training for the purposes of this exercise.

[1] “family guy” is the property of the fox broadcasting company; i am using it here for satirical purposes.
