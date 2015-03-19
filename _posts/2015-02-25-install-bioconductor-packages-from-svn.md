---
layout: post
title: Install bioconductor packages from SVN
date: 2015-02-25 13:50:24.000000000 +01:00
categories: []
tags: []
status: publish
type: post
published: true
---

Due to some issues with the way DEXseq calculates the [log2foldchanges][0] I decided to re-run an analysis with the issue fixed. Since it is not yet in the development branch, an install from svn was needed  - my first!

Firstly the source code was downloaded with:

{% highlight bash %}  
svn co --username readonly --password readonly https://hedgehog.fhcrc.org/bioconductor/branches/RELEASE_3_0/madman/Rpacks/DEXSeq DEXSeq  
{% endhighlight %}

This followed a build to generated a package:

{% highlight bash %}  
R CMD build --no-build-vignettes DEXSeq  
{% endhighlight %}

I had issues with the build process, which failed during the vignette build. As I don't care about it, using `--no-build-vignettes` bypassed the problem. Then it was a simply matter of starting R and installing the package:

{% highlight R %}  
install.packages("DEXSeq_1.12.2.tar.gz", repos=NULL)  
{% endhighlight %}

Simple.

[0]: https://support.bioconductor.org/p/64997/
