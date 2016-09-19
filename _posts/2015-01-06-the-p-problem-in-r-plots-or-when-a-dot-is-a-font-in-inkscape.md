---
layout: post
title: The "p" problem in R plots, or when a dot is a font in inkscape
date: 2015-01-06 10:55:25.000000000 +01:00
categories: []
tags:
- Inkscape
- plots
- R
status: publish
type: post
published: true
---

My graphics/figure workflow generally involves plotting something in R, saving it as a pdf (+png if writing a report with _Rmarkdown_), followed by some manual editing with [Inkscape][0]. Inkscape is a free, as in speech **and** beer, vector graphics editor - think of a non-commercial Illustrator - and it has very very useful since most of the time I don't have access to Illustrator, through a combination of working mostly in Linux and the groups not paying for a license. No matter as Inkscape serves me well, including for personal use.

## Problem

Back to the science. There is a quick with R plots: by default plots save symbols as fonts [dingbats font][1]. This means that when opening a pdf created with R in Inkscape (or Illustrator), points are (correctly) displayed as "p". Even though this is technically correct, it is a nightmare when editing a figure last minute before a presentation.

## Solution

Set set the option `r useDingbats = FALSE` when saving the plot:

{% highlight r %}  
dat <- sample(1:1000,100)  
plot(dat)  
pdf("plot.pdf", useDingbats = FALSE)  
plot(dat)  
dev.off()  
{% endhighlight %}

Or even better, make use of Rprofile for more than [witty R quotes][2], adding to it the line `grDevices::pdf.optionspdf.options(useDingbats = FALSE)`. Like this, you will not have to remember this every time a plot is saved. And if you are like me, this will happen often - both the saving and the forgetting.

[Source1][3]  
[Source2][4]

[0]: https://inkscape.org/
[1]: http://en.wikipedia.org/wiki/Dingbat
[2]: http://movingtothedarkside.wordpress.com/2015/01/05/nerd-up-your-r/
[3]: http://www.inkscapeforum.com/viewtopic.php?f=5&t=7581
[4]: https://support.rstudio.com/hc/communities/public/questions/200664843-Problem-reading-exported-PDFs-in-other-programs
