---
layout: post
title: Rcolorbrewer pallete
date: 2014-12-08 11:04:14.000000000 +01:00
categories: []
tags:
- plots
- R
- RcolorBrewer
status: publish
type: post
published: true
---

I am always looking for a good quality printout of the available palettes in the [RColorBrewer package][0]. I finally decided to take matters on my own hand and create a pdf for myself. Of course it is very easy:

{% highlight R %}  
library(RColorBrewer)  
pdf("~/Pictures/RColorBrewerPalette.pdf")

display.brewer.all()

dev.off()  
{% endhighlight %}

[0]: http://cran.r-project.org/web/packages/RColorBrewer/index.html "RColorBrewer"
