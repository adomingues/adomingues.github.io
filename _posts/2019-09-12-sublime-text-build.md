---
layout: post
title: How style your R code from sublime text
date: 2019-09-12 19:00:00.000000000 +02:00
categories: []
tags:
- R
- sublime text
- software development
status: publish
type: post
published: true
---

Let's face it: we all write ugly R code. Either because we are in a hurry, or we copy pasted from stackoverflow, or our coding style just changed over the course of time, it doesn't really matter: my code is not well formatted and barely adheres to any formatting convention no matter how much I tell myself otherwise. I could do it from an R console or in Rstudio. However the command-line option is too much of an hassle and I use Sublime Text.  

How to do it in `Sublime Text`? 

First of all you need an `R` package such as `styler` installed, then head to  Sublime Text and navigate to `Tools -> Build System  -> New Build System` or `ctrl+shift+p` _Build: New Build_. This should open a new file where you can paste the following code: 

```json
{
    "cmd": [
        "Rscript", "-e",
        "styler::style_file('$file')"
    ],
     "selector": "source.r, source.R",
    "working_dir": "${project_path:${folder}}"
}
```

As you can see this what one would run if calling the command from the terminal, but split into several components. Save the file as whatever you want, in the sublime config directory. Mine (Ubuntu 18.04) was saved as `~/.config/sublime-text-3/Packages/User/styler_file.sublime-build `.

To style your code just call the command pallete, `ctrl+shift+p`, and type the build command name (styler), or `ctrl+b` to list the build options. Press enter and watch the magic happen.

Caveat: it won't work on unsaved files. 

References:
- https://www.sublimetext.com/docs/3/build_systems.html
- http://docs.sublimetext.info/en/latest/reference/build_systems/basics.html
- http://styler.r-lib.org/

