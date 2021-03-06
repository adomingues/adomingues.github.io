---
layout: post
title: Testing for over-representation of anything
date: 2016-02-14 19:02:00.000000000 +01:00
categories: []
tags:
- R
- Bioconductor
- Fisher
status: publish
type: post
published: true
---


Recently I wrote a post on how to test for [chromosome over-representation](http://adomingues.github.io/2015/03/19/chromosome-over-representation-in-DGE/) on a list of genes. The solution, which I thought it was clever at the time, can be simpled to be applied to test if overlap between two lists of genes is significant. Let's use the pasilla data again:


```r
# library("Biobase")
library("pasilla")
library("Biobase")
library("DESeq")

data("pasillaGenes")
```

From that we can select two list of genes to test for overlap significance using my homemade approach (fisher exact test, `fisher.test`):

```r
geneset1 <- sample(rownames(counts(pasillaGenes)), 2500)
geneset2 <- sample(rownames(counts(pasillaGenes)), 3500)

universe <- length(
   unique(rownames(counts(pasillaGenes)))
   )

common <- length(
   intersect(
      unique(geneset1),
      unique(geneset2)
      )
   )


mat <- matrix(
   c(
      universe - length(union(geneset1, geneset2)),
      length(setdiff(geneset1, geneset2)),
      length(setdiff(geneset2, geneset1)),
      length(intersect(geneset1, geneset2))
      ),
   nrow=2
   )

fr <- fisher.test(mat, alternative="greater")
fr
```

Since this a random set of genes, it is not surprising that there is no overlap. This works fairly well and it could even be wrapped in a nice function. Yep, someone else has done [it](http://rpackages.ianhowson.com/bioc/GeneOverlap/man/GeneOverlap.html).


# GeneOverlap

Amongst other things, including visualization of overlaps, this package has a  great function, `testGeneOverlap`, which uses an object created with `newGeneOverlap`, that does exactly the above:


```r
library(GeneOverlap)
overl <- newGeneOverlap(
   unique(geneset1),
   unique(geneset2),
   genome.size=universe)

overl <- testGeneOverlap(overl)
print(overl)
```

```
## Detailed information about this GeneOverlap object:
## listA size=2500, e.g. FBgn0031233 FBgn0035504 FBgn0050468
## listB size=3500, e.g. FBgn0035004 FBgn0085756 FBgn0036820
## Intersection size=583, e.g. FBgn0015805 FBgn0038159 FBgn0052354
## Union size=5417, e.g. FBgn0031233 FBgn0035504 FBgn0050468
## Genome size=14470
## # Contingency Table:
##      notA  inA
## notB 9053 1917
## inB  2917  583
## Overlapping p-value=0.87
## Odds ratio=0.9
## Overlap tested using Fisher's exact test (alternative=greater)
## Jaccard Index=0.1
```

And the results is the same. However, `GeneOverlap` also outputs the results of a few more tests that can be quite useful:

> The Fisher’s exact test also gives an odds ratio which represents the strength of association. If an odds ratio is equal to or less than 1, there is no association between the two lists. If the odds ratio is much larger than 1, then the association is strong. The class also calculates the Jaccard index which measures the similarity between two lists. The Jaccard index varies between 0 and 1, with 0 meaning there is no similarity between the two and 1 meaning the two are identical.

The explanations are also quite nice for beginners. Great Bioconductor package.
