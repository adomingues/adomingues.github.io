---
layout: post
title: Finding the closest element to a number in a vector
date: 2015-09-24
categories: []
tags:
- R
status: publish
type: post
published: true
---


A colleague came to my office the other day with an interesting question:

> Is there a way in R to find the closest number to X in a list?

I knowing full well the power the power of R, I naturally said that surely there is such a function, but I have never used it. So I set out to find it because I am curious. It turns out there is not an of the shelf `closest` function. There are however a few solution out there which I have collected and are bellow. To top it off there is a comparison of how fast each solution is.

# solution 1
[Source](https://stat.ethz.ch/pipermail/r-help/2008-July/167226.html)



```r
x=c(1:10^6)
your.number=90000.43
which(abs(x-your.number)==min(abs(x-your.number)))
```

```
## [1] 90000
```

# solution 2
Same source as before.


```r
which.min(abs(x-your.number))
```

```
## [1] 90000
```


# solution 3
From [here](http://stackoverflow.com/questions/20133344/find-closest-value-in-a-vector-with-binary-search). It requires `data.table`


```r
install.packages("data.table")
```

```
## Installing package into '/home/adomingu/R/x86_64-pc-linux-gnu-library/3.2'
## (as 'lib' is unspecified)
```

```
## Error in contrib.url(repos, type): trying to use CRAN without setting a mirror
```

```r
library(data.table)
dt = data.table(x, val = x) # you'll see why val is needed in a sec
setattr(dt, "sorted", "x")  # let data.table know that w is sorted
setkey(dt, x) # sorts the data

# binary search and "roll" to the nearest neighbour
# In the final expression the val column will have the you're looking for.
dt[J(your.number), roll = "nearest"]
```

```
##        x   val
## 1: 90000 90000
```

# Speed comparison


```r
## time:
# solution1
system.time(which(abs(x-your.number)==min(abs(x-your.number))))
```

```
##    user  system elapsed 
##   0.024   0.020   0.043
```

```r
# solution2
system.time(which.min(abs(x-your.number)))
```

```
##    user  system elapsed 
##   0.008   0.004   0.012
```

```r
# solution3
system.time(dt[J(your.number), roll = "nearest"])
```

```
##    user  system elapsed 
##   0.000   0.000   0.001
```

To my surprise the base R functions perform pretty well, though in really large datasets `data.table` is worth a punt.
