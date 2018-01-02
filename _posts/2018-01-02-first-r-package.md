---
layout: post
title: My first R package - some notes
date: 2018-01-02 14:00:00.000000000 +02:00
categories: []
tags:
- R
- string-manipulation
- package
- software development
status: publish
type: post
published: true
---

I have been toying with the idea of making an R package for sometime. To me this is the natural step after been an `R` user for some years now. Though I had some ideas they all sounded either a bit too over-complicated for a starting package, or not useful enough. In ideal world, and for me personally, I would write a package with a single function, that I could use, to learn the ropes. 

The perfect starting idea came via a twitter [exchange](https://twitter.com/keyboardpipette/status/943110804819382273) over the simplicity and usefulness of `base::make.names`. It turns out that being simple also means it can have some functionality wrapped around it. And this is how [cleaneRnames](https://github.com/adomingues/cleaneRnames) started. The very simple functionality and goal of the package are described in the [github rego](https://github.com/adomingues/cleaneRnames), and this blog post serves only to add some notes and thoughts about the process of actually making the package.   

# Making the package
## skeleton
Installed [usethis](https://github.com/r-lib/usethis) to automate some of the boring tasks. I was incredibly straightforward and quick. 

```r
devtools::install_github("r-lib/usethis")
library(usethis)
setwd("/home/adomingu/sandbox")
pkg_dir <- file.path(".", "cleaneRnames")
create_package(pkg_dir)
```

I choose the license "MIT" because straight and to the point. I am still unsure if this is the right one but it will do for now. 

```r
use_mit_license("António Domingues")
```

The readme was also created using `usethis` which ads some template sentence to get one started. 

```r
use_readme_md()
```

And set-up of the GH repo:

```r
use_git()
```

## The code

Initially I started with opening the `Rstudio` project created by `usethis`, but quickly changed to the more familiar environment of `sublime text`. I saw no point of using `Rstudio`, and IDE, instead of the code editor where I usually live.

Next step was to actually write the code for it. This was ok, but doing it made me think about what was being accomplished here, how to do, some should it be backwards compatible etc. I even started looking into extra functionality, that though nice to have, it was not strictly necessary. In sum, a good exercise in knowing when to stop and what is "good enough".  

## Build 

Karl Broman's [R package primer](http://kbroman.org/pkg_primer/) was very useful at this point and presented two options to build the package:

- command line
- inside `R`

I tested both and they both worked fine. I ended-up doing it inside `R` because of inertia, but if/when I start writing my own packages I will automate the process using a bash script. 

```r
library(devtools)
setwd(pkg_dir)
build()
install()
```

## Documentation

And then we create the documentation.

```r
document()
```

I must say that preparing the docs was where I took the longest, but also the most important. Without documentation no one, including myself, will use the package. Choosing the language/terms to be used was also weird for me not coming from programming/CS background, and wanting to strike a balance between technically correct but understandable by new users. Anyway, this was my first package and I am happy with "good enough". I will definitely repeat the experience.  
