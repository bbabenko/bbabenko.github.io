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

seems to work ok, but it’s a bit slow. if your computer has a multi-core cpu, it’d be nice to leverage all the cores. python threads won’t help you with this, though, because of the GIL (global interpreter lock). luckily, there’s a handy python module called multiprocessing – it effectively spawns multiple python interpreters for you, e.g. one for each core, so that you can distribute your computation. see more here: https://docs.python.org/2/library/multiprocessing.html.

to work up to using multiprocessing, let’s first stick to a single process, but split the computation into 10 chunks:

```python
# single process, using map
ns = np.array([n/10]*10)
rng = np.random.RandomState(0)
in_circle = np.array(map(partial(sample_and_count, rng), ns))
print in_circle
print 'pi ~= {:.10f}'.format(float(4*in_circle.sum())/ns.sum())

# prints out:
# [785057 785849 785758 785431 785483 786147 785751 784570 785366 785661]
# pi ~= 3.1420292000
```

great, we still get the same answer as above. now let’s try with multiprocessing. we should get the same answer as before because we explicitly setting the RNG seed to 0.

```python
# multiprocessing
ns = np.array([n/10]*10)
pool = mp.Pool(10)
rng = np.random.RandomState(0)
in_circle = np.array(pool.map(partial(sample_and_count, rng), ns))
print in_circle
print 'pi ~= {:.10f}'.format(float(4*in_circle.sum())/ns.sum())

# prints out:
# [785057 785057 785057 785057 785057 785057 785057 785057 785057 785057]
# pi ~= 3.1402280000
```

uh-oh… we didn’t get the same answer. the reason this happens is that behind the scenes, the multiprocessing package pickles all the arguments to the function we’re calling and sends the results to each of the processes it spawned. that means the RNG gets a seed of 0, and then gets copied to each process in the pool. each process then runs the exact same thing – this is absolutely not what we want, not only is our answer different from the single process version, the answer is actually worse because we effectively sample the same points multiple times!

one simple solution is to generate an RNG per process, each with a different seed:

```python
# multiprocessing done right
def sample_and_count_2(args):
    rng, n = args
    points = rng.rand(n,2)*2-1
    in_circle = l2norm(points) < 1.0
    return np.sum(in_circle)

ns = np.array([n/10]*10)
pool = mp.Pool(10)
rngs = [np.random.RandomState(i) for i in range(10)]
in_circle = np.array(pool.map(sample_and_count_2, zip(rngs,ns)))
print in_circle
print 'pi ~= {:.10f}'.format(float(4*in_circle.sum())/ns.sum())

# prints out
# [785057 785665 784178 785527 784543 785646 785369 785638 785447 785098]
# pi ~= 3.1408672000
```

obviously you’ll no longer get the same answer as the single process version, but at least the result is deterministic/reproducible and you’re no longer sampling the same points in each process.

does anyone know of any other “recommended” ways of getting around this?
