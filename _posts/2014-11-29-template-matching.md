---
title:  "template matching basics"
date:   2014-11-29
---

i saw a [quora question](https://www.quora.com/How-do-I-locate-the-centers-of-the-beads-in-this-video) float by my feed where someone wanted to detect circular beads in a microscope image.  it looked like a fun little problem, so i decided to take a crack.

normally for detection problems i am all for using a learning-based approach, i.e. label a few examples by hand, train a detector (boosting, random forest, deep learning, whatever works).  but this particular problem seemed simple enough for some good old image processing basics.

here’s the original image:

![](/assets/posts/bubbles/fig1.png)

the first thing i noticed is that the background illumination is uneven.  there is a simple trick to fix this: you blur the image with a wide gaussian kernel to get the background isolated:

![](/assets/posts/bubbles/fig2.png)

then you subtract the background from the original image, and you get this:

![](/assets/posts/bubbles/fig3.png)

some of the beads in the center are a bit faded, but overall the effects of illumination got cleaned away nicely.  this step is probably not strictly required, though i found it much easier to get template matching to work with this sort of normalization.  next, i cropped out one of the beads to use as a template:

![](/assets/posts/bubbles/fig4.png)

the final step is to run template matching (normalized cross correlation), and find the peaks in the response (aka non-maximal suppression), getting you the rough locations of the beads:

![](/assets/posts/bubbles/fig5.png)

results are not perfect – some beads on the sides of the image didn’t get detected, and the locations are probably not perfectly in the center of each bead, but depending on the application, this may suffice.

template matching is pretty brittle, so i wouldn’t hesitate to upgrade to a learning-based approach if the problem got a little more exotic, but there is a time and a place for something simple!

here’s the code:

```python
# see this quora post: http://www.quora.com/How-do-I-locate-the-centers-of-the-beads-in-this-video
import cv2
import numpy as np
from matplotlib import pyplot as plt
from PIL import Image
from skimage.feature import peak_local_max, match_template

blur = lambda img, sigma: cv2.GaussianBlur(
    img, (0, 0), sigma, None, sigma, cv2.BORDER_DEFAULT)
img = np.array(Image.open('beads.png'))
img_float = img.astype('float32')
img_background = blur(img_float, 10)
img_clean = img_float - img_background
template = img_clean[344:344+21,166:166+21:]

# opencv's template matching appears to
# be much faster (around 6x)
if False:
    xcorr = np.squeeze(match_template(img_clean, template))
else:
    xcorr = cv2.matchTemplate(
        img_clean, template, cv2.TM_CCORR_NORMED)

peaks = peak_local_max(xcorr, threshold_abs=0.5, min_distance=3)

plt.clf()
plt.imshow(img)
# peaks will give us locations of the upper left
# corner of the template window, but we want
# the center, so we'll shift the results by
# half the template size
peaks += template.shape[0]/2
plt.plot(peaks[:,1], peaks[:,0], 'rx')
plt.autoscale(tight=True)
```
