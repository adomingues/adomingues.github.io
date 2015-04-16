---
layout: post
title: Painless installation of R packages from source
date: 2015-4-16 10:32:22.000000000 +01:00
categories: []
tags:
- R
- devtools
status: publish
type: post
published: true
---

I was minding my own business trying to add labels to a line plot in `ggplot2`. Then I saw that the package [directlabels](http://directlabels.r-forge.r-project.org/) would solve all my problems with one single line of code. I proceed to install it using `install.packages("directlabels", repo="http://r-forge.r-project.org")`. Sadly:

>package `directlabels` is not available (for R version 3.1.3)

Usually, I would download the source code, then use `install.packages()` from source. But this still did not work due some dependency issues. Thankfully there is the package `devtools` to help:

{% highlight r %}
library(devtools)
install_url("http://cran.r-project.org/src/contrib/directlabels_2013.6.15.tar.gz")
library(directlabels)
{% endhighlight %}

As simple as that. It also allows installation from github repos. 

