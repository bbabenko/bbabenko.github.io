---
title:  "moving my website and blog to github"
date:   2017-09-01
---

i finally bit the bullet and moved and website and blog to [github pages](https://pages.github.com/).  all in all, it was fairly straight forward, and the jekyll backend offers a lot of nice features for technically blog writing:

1. you can write your pages in markdown (though you don't have to, you can use html if you want)

2. you can include blocks of code trivially using markdown's ``` syntax:

    ```python
    import numpy as np

    def blog(**kwargs):
        return str(4+5)
    ```

3. you can use latex for math:

    $$ \sigma _{x}\sigma _{p}\geq {\frac {\hbar }{2}} $$

    to set this up, i followed [this write-up](http://haixing-hu.github.io/programming/2013/09/20/how-to-use-mathjax-in-jekyll-generated-github-pages/) (though i didn't need to change the markdown backend from github's default)

4. you can inherit from a template, and easily overwrite portions.  e.g., i started with the [minima](https://github.com/jekyll/minima) template and i add my tweaks onto that.
you can, of course, see how i set everything up on https://github.com/bbabenko/bbabenko.github.io.
