---
layout: post
title: Nerd up your R
date: 2015-01-05 16:07:19.000000000 +01:00
categories: []
tags:
- fortunes
- R
status: publish
type: post
published: true
---

I probably spend most of my working day in an R terminal, or at least I start R often enough. Now I [saw in a blog I follow][0] a way to entertain and instruct in equal measure everytime that little statistical box is opened: [fortunes][1]. This is a very simple package that collected and now shows citations related to R (and statistics in R). Some examples (and as of v1.5.2 there are 360 citations to choose from):

> Release 1.0.0  
> (silence)  
> Wow! Thank you! [...] If I am allowed to ask just one question today: How do  
> you fit 48 hours of coding in an ordinary day? Any hints will be appreciated  
> ... :-)  
> -- Detlef Steuer (on 2000-02-29)  
> R-help (February 2000)
> 

or

> Perhaps one is the real forge and the other is a forgery? Or a forge-R-y? I'll  
> get my coat...  
> -- Barry Rowlingson (on the question whether http://www.RForge.net/ or  
> http://R-Forge.R-project.org/ is the official forge server)  
> R-help (April 2007)
> 

## Set-up

I have the following lines it in my `~.Rprofile` (think of .bashrc but for R):

{% highlight r %}  
library("fortunes")  
fortune()  
{% endhighlight %}

This ensures each time I start R I am greeted by a nice little message to brighten my day:

![my R start screen](assets/R_fortunes.png)

## How to install

I had some problems installing from _cran_, so went with `install_url` function of devtools which makes the process of installing from source easier when one does not need to download (and keep a copy of) a package source:

{% highlight r %}  
library("devtools")  
install_url("http://cran.r-project.org/src/contrib/fortunes\_1.5-2.tar.gz") #remember to use the link for the latest package version  
{% endhighlight %}

And it is done!

[0]: http://ygc.name/2015/01/05/learn-r-wisdom-with-fortunes/
[1]: http://cran.r-project.org/web/packages/fortunes/index.html
