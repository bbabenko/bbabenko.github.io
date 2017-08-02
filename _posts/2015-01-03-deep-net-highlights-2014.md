---
title:  "deep net highlights from 2014"
date:   2015-01-03
categories: "deep learning"
---


looking back at all the literature that was published on the topic of deep learning in 2014 is quite overwhelming.  at times it has felt that nearly every week a new paper from some heavyweights was being circulated via arxiv.  trying to do a review of all that work would be a fool’s errand, but i thought it might be worthwhile to summarize some of the threads that i found particularly interesting.  below i group papers by broader topic.

## exploring different architectures

while deep nets promise to free you from designing low and mid level features by hand, designing the architecture for a network is still somewhat of a black art (which is still quickly evolving).  a number of papers this year explored alternatives to the popular “alex-net” architecture:

### network in network, lin et al.

a typical convolutional net consists of convolutional layers (i.e. a number of small filters is applied to the input), following by a non-linearity (e.g. sigmoid, rectified linear unit, etc), followed by max pooling (this is repeated a few times, and typically topped off with a couple fully connected layers).  the authors of “network in network” argue that the linear filtering operation in the convo layers is not expressive enough, leading to a necessity of many layers stacked on top of each other.  they propose that instead of a linear filter, one can use a small multi-layered perceptron that slides over the input channels.  the following diagram illustrates such a layer:

![My helpful screenshot](/assets/posts/dl-2014/network_in_network.png)
