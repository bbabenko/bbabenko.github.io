---
title:  "when hogs py"
date:   2013-07-28
---

(get it? when pigs fly? ok then…)

histograms of oriented gradients [1], or HOG, are a very popular image feature in computer vision.  the recipe is pretty straight forward: the image is divided into (usually 8x8) cells, for each cell you compute a (usually 9 bin) gradient orientation histogram.  then there’s a funky normalization step where you group cells into blocks (typically a block is 2x2 cells or 3x3 cells), and your descriptor consists of going through each block and normalizing the histogram of each cell in that block by the block’s magnitude (i.e. each cell is represented multiple times in the final descriptor; the paper contains a much better explanation).

the other day i was looking to try out hog in python, and it turned out surprisingly difficult to find a good implementation.  all in all i found two: one in opencv and another in skimage.  i decided to compare the two.

first there is the issue of documentation.  opencv documentation for python is…. lacking.  the only way you can figure out that the HOG stuff is even accessible via python is by googling around.  to figure out what the parameters were i had to glance through this code.  skimage is definitely better in this department.

next i did a sanity check: i wanted to make sure the dimensionally of the output is correct for both.  with the parameters i chose, i should have ended up with:

9 orientations X (4 corner blocks that get 1 normalization + 6x4 blocks on the edges that get 2 normalizations + 6x6 blocks that get 4 normalizations) = 1764.  

finally, i timed these bad boys.  results are below:

```python
In [1]: import numpy as np
In [2]: from cv2 import HOGDescriptor
In [3]: from skimage.feature import hog
In [4]: image = (np.random.rand(64,64)*255).astype('uint8')
In [5]: # OPENCV
In [6]: opencv_hog=HOGDescriptor((64,64), (16,16), (8,8), (8,8), 9)
In [7]: h1 = opencv_hog.compute(image)
In [8]: h1.shape
Out[8]: (1764, 1)
In [9]: # SKIMAGE
In [10]: h2 = hog(image, orientations=9, pixels_per_cell=(8, 8), cells_per_block=(2, 2))
In [11]: h2.shape
Out[11]: (1764,)
In [12]: %timeit -n100 -r1 opencv_hog.compute(image)
100 loops, best of 1: 204 us per loop
In [13]: %timeit -n100 -r1 hog(image)
100 loops, best of 1: 6.27 ms per loop
```

as you can see, the dimensionality checks out.  also, looks like opencv is about 30x faster.  the [skimage implementation](https://github.com/scikit-image/scikit-image/blob/master/skimage/feature/_hog.py) is written in pure python (i.e. not cython), so a 30x difference is about what one would expected between a python and c++ implementations.

now, are the outputs of these two implementations the same?  they aren’t, and i’m not motivated enough to sift through the code and figure out what the differences are or whether one is more correct than the other :) (if you caught the sift pun, then kudos to you).

conclusion: if you are ok with no documentation, go with opencv.  if you’re looking for code you can easily debug and extend, go with skimage.

[1] [dalal and triggs](http://hal.archives-ouvertes.fr/docs/00/54/85/12/PDF/hog_cvpr2005.pdf)
