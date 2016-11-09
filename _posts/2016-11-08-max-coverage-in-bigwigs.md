---
layout: post
title: Function to find maximal coverage in multiple bigwigs
date: 2016-11-08 20:32:22.000000000 +01:00
categories: []
tags:
- R
- gviz
- visualization
status: publish
type: post
published: true
---

I really like the package Gviz to prepare figures for presentations and publications (I have used it in [B](http://d2ni3bh4dzb2ig.cloudfront.net/content/embojnl/early/2014/12/18/embj.201490061/F4.large.jpg?width=800&height=600&carousel=1) with some tidying up in inskape).

It is a fantastic visualization package, but the time and effort that it takes to get the figures *just right* is a little too much for my daily data inspection/visualization tasks. An example of this is when plotting coverage tracks; by default axis of each panel are scaled independently which makes visualization tricky. So I created a little function that will loop over a list of `BigWig` tracks, find the maximal coverage in each chromosome, and return the max of all tracks. This value can no be used as `ymax` in plots with multiple tracks.

```r
  ## calculate the max values
   maxCovBw <- function(gr, myChr) {
      max_cov <- max(gr[seqnames(gr) %in% myChr,]$score)
      return(max_cov)
   }

   maxCovFiles <- function(files, chrs){
      reps_gr <- lapply(files, rtracklayer:::import)
      for(i in seq_along(chrs)){
         myChr = chrs[i]
         print(myChr)
         max_coverage <- c()
         max_cov <- round(
            max(
               sapply(reps_gr, maxCovBw, myChr=myChr)
            )
         , 0)
         max_coverage[myChr] <- max_cov 
      }
      return(data.frame(max_coverage))
   }

   max_cov <- maxCovFiles(list(bw1_file, bw2_file), seqinfo(bw1)$seqnames)
```

And why chromosomes you ask. Because:
1. Gviz plots are chromosome-centric, that is one needs to specify the chromosome for each track, and all tracks in a plot must be in the same location.
2. The particular plot that I am currently working on is a overview of a full contig, so I don't zoom in to a particular range in that contig.

A reproducible example might appear in the future.
