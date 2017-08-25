---
title:  "moving my website and blog to github"
date:   2017-08-25
use_math: true
---

i finally bit the bullet and moved and website and blog to [github pages](https://pages.github.com/).  all in all, it was fairly straight forward, and the jekyll backend offers a lot of nice features for technical blog writing:

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

4. you can inherit from a template, and easily overwrite/overload portions.  e.g., i started with the [minima](https://github.com/jekyll/minima) template and i added my tweaks onto that.
you can, of course, see how i set everything up on https://github.com/bbabenko/bbabenko.github.io.

5. you can write an entire post in a python/jupyter notebook, and easily convert that into a markdown file to be published on your blog.  the code blocks and latex will get rendered as expected.  to do this, you can just run the `jupyter nbconvert SOME_NOTEBOOK.ipynb --to markdown` command; you'll need to make some very minor adjustments to the .md file that is rendered (mostly just fixing the paths to figures). this is a pretty killer feature/workflow, in my opinion.  see [this page](https://briancaffey.github.io/2016/03/14/ipynb-with-jekyll.html) for more info.

migrating old blog posts required some elbow grease (and i didn't migrate all of them), but all in all i am happy with the result!
