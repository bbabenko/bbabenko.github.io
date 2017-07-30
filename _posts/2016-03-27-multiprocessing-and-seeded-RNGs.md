---
title:  "multiprocessing and seeded RNGs"
date:   2016-03-27
categories: multiprocessing python rng
---

a while back i ran across a pretty subtle, devious bug and wanted to share. the bug has to do with multiprocessing and seeded random number generators. i’ll explain via a toy example.

suppose you want to estimate pi via monte carlo (see http://www.eveandersson.com/pi/monte-carlo-circle). you can do this by sampling points in the [-1,1]x[-1,1] square and then seeing how many fall inside the unit circle. to get an estimate of pi you then use the simple formula 4*(# points that fell into unit circle) / (# total points you drew) – i won’t bore you with the algebra to derive this formula.

one thing i like to do whenever i use a random number generator is to explicitly set the seed. this ensures that my experiment is reproducible.

let’s try this in python:

```python
import multiprocessing as mp
import numpy as np
from functools import partial
from matplotlib import pyplot as plt
from scipy.linalg import norm

def l2norm(v):
    return np.sqrt(np.sum(v**2, axis=1))

def sample_and_count(rng, n):
    points = rng.rand(n,2)*2-1
    in_circle = l2norm(points) < 1.0
    return np.sum(in_circle)

# single process
n = 10000000
rng = np.random.RandomState(0)
in_circle = sample_and_count(rng, n)
print 'pi ~= {:.10f}'.format(float(4*in_circle)/n)

# prints out:
# pi ~= 3.1420292000
```
