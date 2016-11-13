---
layout: post
title: Function to find maximal coverage in multiple bigwigs II
date: 2016-11-13 23:41:22.000000000 +01:00
categories: []
tags:
- R
- gviz
- visualization
status: publish
type: post
published: true
---

[This is an updated version of this [post](http://adomingues.github.io/2016/11/08/max-coverage-in-bigwigs/) with improved functions and a reproducible example]

I really like the package Gviz to prepare figures for presentations and publications (I have used in [B](http://d2ni3bh4dzb2ig.cloudfront.net/content/embojnl/early/2014/12/18/embj.201490061/F4.large.jpg?width=800&height=600&carousel=1) plus some tidying up in inskape). It is a fantastic visualization package, but the time and effort that it takes to get the figures *just right* is a little too much for my daily. An example of this is when plotting coverage tracks; by default axis of each panel are scaled independently which makes visualization tricky. 

let's fist load the packages. `[derfinder](http://bioconductor.org/packages/release/bioc/html/derfinderPlot.html)` will the one containing the example datasets (chosen almost randomly).




{% highlight r %}
library('Gviz')
library('rtracklayer')
library('derfinderData')
files <- dir(system.file('extdata', 'A1C', package = 'derfinderData'),  full.names = TRUE)
names(files) <- gsub('\\.bw', '', dir(system.file('extdata', 'AMY', package = 'derfinderData')))
head(files)
{% endhighlight %}



{% highlight text %}
##                                                                                 HSB113 
## "/home/adomingu/R/x86_64-pc-linux-gnu-library/3.3/derfinderData/extdata/A1C/HSB103.bw" 
##                                                                                 HSB123 
## "/home/adomingu/R/x86_64-pc-linux-gnu-library/3.3/derfinderData/extdata/A1C/HSB114.bw" 
##                                                                                 HSB126 
## "/home/adomingu/R/x86_64-pc-linux-gnu-library/3.3/derfinderData/extdata/A1C/HSB123.bw" 
##                                                                                 HSB130 
## "/home/adomingu/R/x86_64-pc-linux-gnu-library/3.3/derfinderData/extdata/A1C/HSB126.bw" 
##                                                                                 HSB135 
## "/home/adomingu/R/x86_64-pc-linux-gnu-library/3.3/derfinderData/extdata/A1C/HSB130.bw" 
##                                                                                 HSB136 
## "/home/adomingu/R/x86_64-pc-linux-gnu-library/3.3/derfinderData/extdata/A1C/HSB135.bw"
{% endhighlight %}



{% highlight r %}
bw_file1 <- files[1]
bw_file2 <- files[3]
bw1 <- import(files[1])
bw2 <- import(files[2])
{% endhighlight %}

Let's create some genomic regions to look at:


{% highlight r %}
gr1 <- GRanges(
  seqnames=c("chr21", "chr21"),
  ranges=IRanges(c(48055507, 17059283), c(48085155, 17070283)),
  strand=c("+", "-")
  )
gr1
{% endhighlight %}



{% highlight text %}
## GRanges object with 2 ranges and 0 metadata columns:
##       seqnames               ranges strand
##          <Rle>            <IRanges>  <Rle>
##   [1]    chr21 [48055507, 48085155]      +
##   [2]    chr21 [17059283, 17070283]      -
##   -------
##   seqinfo: 1 sequence from an unspecified genome; no seqlengths
{% endhighlight %}

One of these is the gene _PRMT2_, and the other is some random region without an annotated gene. We will see it's usefulness later.

And does the plot look like using the defaults? Let's have a look:



{% highlight r %}
gen = "hg19"
gr2 <- as.data.frame(gr2)
{% endhighlight %}



{% highlight text %}
## Error in as.data.frame(gr2): object 'gr2' not found
{% endhighlight %}



{% highlight r %}
start=gr2[1,2]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'gr2' not found
{% endhighlight %}



{% highlight r %}
end=gr2[1,3]
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'gr2' not found
{% endhighlight %}



{% highlight r %}
gTrack <- BiomartGeneRegionTrack(
  genome = gen,
  chromosome = "chr21",
  start = start,
  end = end,
  name = "ENSEMBL")
{% endhighlight %}



{% highlight text %}
## Error in r[i1] - r[-length(r):-(length(r) - lag + 1L)]: non-numeric argument to binary operator
{% endhighlight %}



{% highlight r %}
dTrack1 <- DataTrack(
  bw_file1,
  type = "l",
  name="track1")
dTrack2 <- DataTrack(
  bw_file2,
  type = "l",
  name="track2")

# call the display function plotTracks
track.list=list(dTrack1, dTrack2, gTrack)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'gTrack' not found
{% endhighlight %}



{% highlight r %}
plotTracks(track.list,from=start,to=end,chromsome="chr21")
{% endhighlight %}



{% highlight text %}
## Error in plotTracks(track.list, from = start, to = end, chromsome = "chr21"): object 'track.list' not found
{% endhighlight %}

Two things become clear, though not immediately:
- the coverage is lower in track2, but since these tracks do not share the same scale a simple glance would convey the wrong message.
- Due to the internals of Gviz, the `y-axis` of both tracks have different front size. This is because they are scaled to the `y-max`. This could be fixed, but makes the figure a bit clunky for presentations.

The way I am solving both these issues is to determine the maximal coverage for both tracks, and then adjust the `y-lim` of both to this value. Initially I was doing this manually, using a `Kent` tool in the terminal, and using the value in R. This is of course time consuming and not adequate if one starts using Gviz on a regular basis, mixing and matching tracks in the plots. 

Becau I had a project that required me to plot the coverage over an entire chomosome rather than a gene, my first programatic solution was to calculate the top coverage for a given chromosome. The basic function `maxCovBw` will take an imported `BigWig` and do this. The `maxCovFiles` function is a wrapper that will do the same but for a list of files. 


{% highlight r %}
## calculate the max values
 maxCovBw <- function(gr, myChr) {
    max_cov <- max(gr[seqnames(gr) %in% myChr,]$score)
    return(max_cov)
 }

 maxCovFiles <- function(bws, chrs){
    for(i in seq_along(chrs)){
       myChr = chrs[i]
       print(myChr)
       max_coverage <- c()
       max_cov <- round(
          max(
             sapply(bws, maxCovBw, myChr=myChr)
          )
       , 0)
       max_coverage[myChr] <- max_cov 
    }
    return(data.frame(max_coverage))
 }

max_cov <- maxCovFiles(list(bw1, bw2), seqnames(seqinfo(bw1)))
{% endhighlight %}



{% highlight text %}
## [1] "chr21"
{% endhighlight %}
We can see that there is only one chromosome in these files:


{% highlight r %}
seqnames(seqinfo(bw1))
{% endhighlight %}



{% highlight text %}
## [1] "chr21"
{% endhighlight %}



{% highlight r %}
seqnames(seqinfo(bw2))
{% endhighlight %}



{% highlight text %}
## [1] "chr21"
{% endhighlight %}
which we can now use to call the function:


{% highlight r %}
max_cov <- maxCovFiles(list(bw1, bw2), "chr21")
{% endhighlight %}



{% highlight text %}
## [1] "chr21"
{% endhighlight %}



{% highlight r %}
max_cov
{% endhighlight %}



{% highlight text %}
##       max_coverage
## chr21         2250
{% endhighlight %}

As I mentioned, this is useful for those situations when I am plotting a full chromosome, but in most situations what is being plotted is a narrow region, for instance, at a gene. Also a chromosome could be represented as a region as `c(0, seqlengths(seqinfo(bw1))`. So let's modify the function:



{% highlight r %}
## calculate the max values
maxCovBw <- function(bw, gr) {
  ovlp <- subsetByOverlaps(bw, gr)
  if (length(ovlp) > 0) {
    print('not empty')
    max_cov <- max(ovlp$score)
  } else {
    print('WARNING: The selected genomic region has no coverage value in the BigWig')
    print('WARNING: Coverage value is arbitrary set to Zero.')
    print('Region:')
    print(gr)
    print('BigWig:')
    print(bw)
    max_cov <- 0 
  }
  print(max_cov)
  return(max_cov)
}

maxCovFiles <- function(bws, gr){
  # bws <- lapply(bws, rtracklayer:::import)
  max_cov <- c()
  for(i in 1:length(gr)){
    my_feat = gr[i, ]
    max_cov[i] <- round(
      max(
        sapply(bws, maxCovBw, gr=my_feat)
      )
      , 2)
  }
  values(gr) <- max_cov 
  return(gr)
}


gr2 <- maxCovFiles(list(bw1, bw2), gr1)
{% endhighlight %}



{% highlight text %}
## [1] "not empty"
## [1] 19.84
## [1] "not empty"
## [1] 10.06
## [1] "WARNING: The selected genomic region has no coverage value in the BigWig"
## [1] "WARNING: Coverage value is arbitrary set to Zero."
## [1] "Region:"
## GRanges object with 1 range and 0 metadata columns:
##       seqnames               ranges strand
##          <Rle>            <IRanges>  <Rle>
##   [1]    chr21 [17059283, 17070283]      -
##   -------
##   seqinfo: 1 sequence from an unspecified genome; no seqlengths
## [1] "BigWig:"
## GRanges object with 153524 ranges and 1 metadata column:
##            seqnames               ranges strand |              score
##               <Rle>            <IRanges>  <Rle> |          <numeric>
##        [1]    chr21   [9481565, 9481639]      * | 0.0299999993294477
##        [2]    chr21   [9496483, 9496557]      * | 0.0299999993294477
##        [3]    chr21   [9543846, 9543920]      * | 0.0299999993294477
##        [4]    chr21   [9551135, 9551209]      * | 0.0299999993294477
##        [5]    chr21   [9583531, 9583605]      * | 0.0299999993294477
##        ...      ...                  ...    ... .                ...
##   [153520]    chr21 [48093192, 48093219]      * |  0.100000001490116
##   [153521]    chr21 [48093220, 48093266]      * | 0.0700000002980232
##   [153522]    chr21 [48093307, 48093323]      * | 0.0299999993294477
##   [153523]    chr21 [48093324, 48093381]      * | 0.0700000002980232
##   [153524]    chr21 [48093382, 48093398]      * | 0.0299999993294477
##   -------
##   seqinfo: 1 sequence from an unspecified genome
## [1] 0
## [1] "WARNING: The selected genomic region has no coverage value in the BigWig"
## [1] "WARNING: Coverage value is arbitrary set to Zero."
## [1] "Region:"
## GRanges object with 1 range and 0 metadata columns:
##       seqnames               ranges strand
##          <Rle>            <IRanges>  <Rle>
##   [1]    chr21 [17059283, 17070283]      -
##   -------
##   seqinfo: 1 sequence from an unspecified genome; no seqlengths
## [1] "BigWig:"
## GRanges object with 120291 ranges and 1 metadata column:
##            seqnames               ranges strand |              score
##               <Rle>            <IRanges>  <Rle> |          <numeric>
##        [1]    chr21   [9699556, 9699630]      * | 0.0599999986588955
##        [2]    chr21   [9825442, 9825443]      * |  0.119999997317791
##        [3]    chr21   [9825444, 9825445]      * |  0.180000007152557
##        [4]    chr21   [9825446, 9825472]      * |  0.239999994635582
##        [5]    chr21   [9825473, 9825490]      * |  0.300000011920929
##        ...      ...                  ...    ... .                ...
##   [120287]    chr21 [48088330, 48088404]      * | 0.0599999986588955
##   [120288]    chr21 [48088479, 48088553]      * | 0.0599999986588955
##   [120289]    chr21 [48090239, 48090313]      * | 0.0599999986588955
##   [120290]    chr21 [48090346, 48090420]      * | 0.0599999986588955
##   [120291]    chr21 [48090683, 48090757]      * | 0.0599999986588955
##   -------
##   seqinfo: 1 sequence from an unspecified genome
## [1] 0
{% endhighlight %}



{% highlight r %}
gr2
{% endhighlight %}



{% highlight text %}
## GRanges object with 2 ranges and 1 metadata column:
##       seqnames               ranges strand |         X
##          <Rle>            <IRanges>  <Rle> | <numeric>
##   [1]    chr21 [48055507, 48085155]      + |     19.84
##   [2]    chr21 [17059283, 17070283]      - |         0
##   -------
##   seqinfo: 1 sequence from an unspecified genome; no seqlengths
{% endhighlight %}

As you can see we now have the top value (which is pretty low) for each of our regions of interest. The coordinates selected contain an edge case for which there is no overlap. I decided to set it to `zero` to avoid breaking things. How does this look like in a plot then?



{% highlight r %}
max_cov=gr2[1,6]
{% endhighlight %}



{% highlight text %}
## Error: subscript contains NAs or out-of-bounds indices
{% endhighlight %}



{% highlight r %}
dTrack1 <- DataTrack(
  bw1,
  type = "l",
  ylim=c(0, max_cov),
  name="track1")
dTrack2 <- DataTrack(
  bw2,
  type = "l",
  ylim=c(0, max_cov),
  name="track2")

# call the display function plotTracks
track.list=list(dTrack1, dTrack2, gTrack)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'gTrack' not found
{% endhighlight %}



{% highlight r %}
plotTracks(track.list,from=start,to=end,chromsome="chr21")
{% endhighlight %}



{% highlight text %}
## Error in plotTracks(track.list, from = start, to = end, chromsome = "chr21"): object 'track.list' not found
{% endhighlight %}

This simple change, `ylim=c(0, max_cov)`, makes massive visual difference. A quick look at the plot and it will immediately be clear that `track1` has higher coverage, and the visuals are now more consistent. This is something that could go into a manuscript dratf without too many changes.

Of course both functions could do with some more input testing and cleaning up, but they are working and a good starting point for a quick plot. One thing that is missing and would be useful is the ability use `bam/sam` as an input as `BigWigs` would become limiting in certain situations. Something to improve once the need arises.   

