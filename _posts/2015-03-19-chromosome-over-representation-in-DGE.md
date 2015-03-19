---
layout: post
title: Testing for chromosome over-representation
date: 2015-12-19 10:32:22.000000000 +01:00
categories: []
tags:
- R
- Bioconductor
- Fisher
- biomaRt
status: publish
type: post
published: false
---

# Chromosome over-representation in DGE

Sometimes I am working on some data and notice certain biases, say for differentially expressed genes appearing to originate more often from a chromosome. Or a factor binding more often to a class of transcripts. In these situations I tend to turn to Fisher's exact test. Here I will put an example of what I do.


## Get some data

For the sake of simplicity I will use data from the Pasilla[1] dataset and run differential gene expression analysis with DESeq2[2] following the vignette's instructions. The `data.table` package is used because I like it's speed and syntax, specially for sub-setting. Since I am still learning all the ins and outs of it, I mix `data.frame` code with it. Whatever works.


{% highlight r %}
library("data.table")
library("pasilla")
library("Biobase")
data("pasillaGenes")
library("DESeq2")
library("biomaRt")
{% endhighlight %}

While loading the `pasillaGenes` R was throwing an error at me:

> Warning: namespace ‘DESeq’ is not available and has been replaced

It turns out that `DESeq` needed to be installed to load the data. No idea why. So let's get us some results:


{% highlight r %}
countData <- counts(pasillaGenes)
colData <- pData(pasillaGenes)[,c("condition","type")]
dds <- DESeqDataSetFromMatrix(
   countData = countData,
   colData = colData,
   design = ~ condition
   )
dds$condition <- relevel(dds$condition, "untreated")
dds <- DESeq(dds)
{% endhighlight %}

## Add chromosome information
Now the fun starts. I created a `data.table` with the results plus gene IDs, And will now added some extra information with biomaRt. I have only added chromosome and biotype, but the amount of information one can add is large. I usually also add gene symbol and description. Very useful to have a quick idea of the function of a particular gene. Adding the full gene location might be useful to later on subset data and easily create a bed file.


{% highlight r %}
res <- as.data.frame(results(dds))
res$ensembl_gene_id <- rownames(results(dds))
res <- data.table(res)

ensembl = useMart("ensembl", dataset = "dmelanogaster_gene_ensembl")
genemap <- getBM( attributes = c("ensembl_gene_id", "chromosome_name", "gene_biotype"),
                  filters = "ensembl_gene_id",
                  values = res$ensembl_gene_id,
                  mart = ensembl)
idx <- match( res$ensembl_gene_id, genemap$ensembl_gene_id )
res$chromosome <- genemap$chromosome_name[ idx ]
res$gene_biotype <- genemap$gene_biotype[ idx ]
{% endhighlight %}

Now let's have a look at down-regulated genes with `padj < 0.1`.


{% highlight r %}
res[padj < 0.1 & log2FoldChange < 0]
{% endhighlight %}



{% highlight text %}
##        baseMean log2FoldChange     lfcSE      stat       pvalue
##   1:   85.00693     -0.7067901 0.2196386 -3.217969 1.291017e-03
##   2:  273.13010     -0.3878740 0.1360678 -2.850593 4.363776e-03
##   3: 1257.88508     -0.3896927 0.1321601 -2.948640 3.191754e-03
##   4:  280.32056     -0.4471012 0.1417352 -3.154483 1.607826e-03
##   5:  141.36961     -1.3409377 0.2076951 -6.456280 1.073079e-10
##  ---                                                           
## 404:  998.56567     -0.4335869 0.1357833 -3.193228 1.406921e-03
## 405:   16.42232     -0.6973109 0.2276632 -3.062906 2.191992e-03
## 406:  373.24468     -0.4988888 0.1441100 -3.461860 5.364557e-04
## 407: 2891.60114     -1.6134844 0.1706094 -9.457183 3.163516e-21
## 408:  973.77088     -0.8325356 0.1332384 -6.248464 4.145082e-10
##              padj ensembl_gene_id chromosome   gene_biotype
##   1: 2.007727e-02     FBgn0000079         2R protein_coding
##   2: 5.230444e-02     FBgn0000244         3R protein_coding
##   3: 4.105429e-02     FBgn0000256         2L protein_coding
##   4: 2.389391e-02     FBgn0000286         2L protein_coding
##   5: 1.045641e-08     FBgn0000406         2L protein_coding
##  ---                                                       
## 404: 2.158653e-02     FBgn0261545         2R protein_coding
## 405: 3.079189e-02     FBgn0261546         NA             NA
## 406: 9.950931e-03     FBgn0261547         3L protein_coding
## 407: 1.106943e-18     FBgn0261552         3R protein_coding
## 408: 3.710330e-08     FBgn0261560         2L protein_coding
{% endhighlight %}

Most of these appear to be coding genes. Not a great surprise. But...  Oh! There are a lot of genes located in chromosome 2L. Is this a coincidence? Let us test it.


## Test for over-representation

### Chromosome 3L

{% highlight r %}
chrom="3L"
all <- res$ensembl_gene_id
hits <- res[padj < 0.1 & log2FoldChange < 0]$ensembl_gene_id
hits_in_chr <- length(res[chromosome == chrom & padj < 0.1 & log2FoldChange < 0]$ensembl_gene_id)
genes_in_chr <- length(res[chromosome == chrom]$ensembl_gene_id)
hits_total <- length(hits)
genes_total <- length(all)
{% endhighlight %}

Expected number of hits in chromosome 3L would be 71.8440912 and we have 61. Is this significant? I will use Fisher's exact test, as applied [here](https://www.biostars.org/p/102946/) and [here](http://cgrlucb.wikispaces.com/Functional+Enrichment+Analysis) for very similar problems.

Firstly I construct a table/matrix with the events and from that calculated Fisher's Exact test, or the probability of having more genes in chromosome 3L than expected by chance. There are more details [here](http://cgrlucb.wikispaces.com/Functional+Enrichment+Analysis).


{% highlight r %}
mat <- matrix(
   c(
      hits_in_chr,
      genes_in_chr-hits_in_chr,
      hits_total-hits_in_chr,
      genes_total-hits_total-genes_in_chr+hits_in_chr
      ),
      nrow=2,
      ncol=2)
mat
{% endhighlight %}



{% highlight text %}
##      [,1]  [,2]
## [1,]   61   347
## [2,] 2487 11575
{% endhighlight %}



{% highlight r %}
fr <- fisher.test(mat, alternative="greater")
fr
{% endhighlight %}



{% highlight text %}
## 
## 	Fisher's Exact Test for Count Data
## 
## data:  mat
## p-value = 0.935
## alternative hypothesis: true odds ratio is greater than 1
## 95 percent confidence interval:
##  0.6400274       Inf
## sample estimates:
## odds ratio 
##  0.8181856
{% endhighlight %}

So there is a p-value of 0.9349979 and we can reject the hypothesis that there is enrichment for or genes in 3L. My eyes were seeing patterns where this is none. Also, since I was testing only for over-representation, or enrichment, the option `alternative="greater"` was used in the test. Other options are available.


### For all chromosomes

Is this this the case for any of the other chromosome? I will construct a function, and then loop over the chromosomes.


The function `chrEnrichment` is minimal and it only tests for down-regulated gens, but could be easily extended to add other arguments. It could also be used to test for biases in gene biotype.



{% highlight r %}
chrEnrichment <- function(chr, df){
   # Test for chromosome overpresentation in DESe2 results
   all <- df$ensembl_gene_id
   hits <- df[padj < 0.1 & log2FoldChange < 0]$ensembl_gene_id

   hits_in_chr <- length(res[chromosome == chr & padj < 0.1 & log2FoldChange < 0]$ensembl_gene_id)
   genes_in_chr <- length(res[chromosome == chr]$ensembl_gene_id)
   hits_total <- length(hits)
   genes_total <- length(all)

   mat <- matrix(
      c(
         hits_in_chr,
         genes_in_chr-hits_in_chr,
         hits_total-hits_in_chr,
         genes_total-hits_total-genes_in_chr+hits_in_chr
         ),
         nrow=2,
         ncol=2)
   fr <- fisher.test(mat, alternative="greater")
   df <- data.frame(
      chromosome = chromosome,
      observed = hits_in_chr,
      expected = round(hits_total * genes_in_chr / genes_total, 1),
      odds.ratio= fr$estimate[["odds ratio"]],
      pvalue = fr$p.value
      )
   return(df)
}


chromosomes <- unique(na.omit(res[!like(chromosome,"Zv9")])$chromosome)
l_res <- list()
for (chromosome in chromosomes){
   l_res[[chromosome]] <- chrEnrichment(chromosome, res)
}

table_fisher <- rbindlist(l_res)
table_fisher[,FDR:=p.adjust(pvalue, method='bonferroni'),] # add FDR
{% endhighlight %}



{% highlight text %}
##    chromosome observed expected odds.ratio     pvalue       FDR
## 1:         2R       91     78.1  1.2197190 0.05874683 0.4112278
## 2:         3L       61     71.8  0.8181856 0.93499788 1.0000000
## 3:         2L       86     70.2  1.2958104 0.02276513 0.1593559
## 4:         3R       95     90.2  1.0719388 0.29753414 1.0000000
## 5:          X       46     58.6  0.7523146 0.97261113 1.0000000
## 6:          4        0      2.3  0.0000000 1.00000000 1.0000000
## 7:       YHet        0      0.4  0.0000000 1.00000000 1.0000000
{% endhighlight %}



{% highlight r %}
setkey(table_fisher, FDR) # will also sort by pvalue
{% endhighlight %}

Since I am testing for all chromosomes, I have also calculated the adjusted p-value (bonferroni) to be on the safe side.

And what does Fisher say?


{% highlight r %}
table_fisher
{% endhighlight %}



{% highlight text %}
##    chromosome observed expected odds.ratio     pvalue       FDR
## 1:         2L       86     70.2  1.2958104 0.02276513 0.1593559
## 2:         2R       91     78.1  1.2197190 0.05874683 0.4112278
## 3:         3L       61     71.8  0.8181856 0.93499788 1.0000000
## 4:         3R       95     90.2  1.0719388 0.29753414 1.0000000
## 5:          X       46     58.6  0.7523146 0.97261113 1.0000000
## 6:          4        0      2.3  0.0000000 1.00000000 1.0000000
## 7:       YHet        0      0.4  0.0000000 1.00000000 1.0000000
{% endhighlight %}

That we have a pretty much random distribution of down-regulated genes in the fly chromosomes.

## References
[1]: http://bioconductor.org/packages/release/data/experiment/html/pasilla.html
[2]: http://www.bioconductor.org/packages/release/bioc/html/DESeq2.html



{% highlight r %}
sessionInfo()
{% endhighlight %}



{% highlight text %}
## R version 3.1.3 (2015-03-09)
## Platform: x86_64-pc-linux-gnu (64-bit)
## Running under: Ubuntu 14.04.2 LTS
## 
## locale:
##  [1] LC_CTYPE=en_US.UTF-8       LC_NUMERIC=C              
##  [3] LC_TIME=de_DE.UTF-8        LC_COLLATE=en_US.UTF-8    
##  [5] LC_MONETARY=de_DE.UTF-8    LC_MESSAGES=en_US.UTF-8   
##  [7] LC_PAPER=de_DE.UTF-8       LC_NAME=C                 
##  [9] LC_ADDRESS=C               LC_TELEPHONE=C            
## [11] LC_MEASUREMENT=de_DE.UTF-8 LC_IDENTIFICATION=C       
## 
## attached base packages:
## [1] stats4    parallel  methods   stats     graphics  grDevices utils    
## [8] datasets  base     
## 
## other attached packages:
##  [1] biomaRt_2.22.0            DESeq2_1.6.3             
##  [3] RcppArmadillo_0.4.650.1.1 Rcpp_0.11.5              
##  [5] GenomicRanges_1.18.4      GenomeInfoDb_1.2.4       
##  [7] IRanges_2.0.1             S4Vectors_0.4.0          
##  [9] Biobase_2.26.0            BiocGenerics_0.12.1      
## [11] pasilla_0.5.1             data.table_1.9.4         
## [13] knitr_1.9                
## 
## loaded via a namespace (and not attached):
##  [1] acepack_1.3-3.3      annotate_1.44.0      AnnotationDbi_1.28.1
##  [4] base64enc_0.1-2      BatchJobs_1.5        BBmisc_1.9          
##  [7] BiocParallel_1.0.3   bitops_1.0-6         brew_1.0-6          
## [10] checkmate_1.5.1      chron_2.3-45         cluster_2.0.1       
## [13] codetools_0.2-11     colorspace_1.2-6     DBI_0.3.1           
## [16] DESeq_1.18.0         digest_0.6.8         evaluate_0.5.5      
## [19] fail_1.2             foreach_1.4.2        foreign_0.8-63      
## [22] formatR_1.0          Formula_1.2-0        genefilter_1.48.1   
## [25] geneplotter_1.44.0   ggplot2_1.0.1        grid_3.1.3          
## [28] gtable_0.1.2         Hmisc_3.15-0         iterators_1.0.7     
## [31] lattice_0.20-30      latticeExtra_0.6-26  locfit_1.5-9.1      
## [34] MASS_7.3-39          munsell_0.4.2        nnet_7.3-9          
## [37] plyr_1.8.1           proto_0.3-10         RColorBrewer_1.1-2  
## [40] RCurl_1.95-4.5       reshape2_1.4.1       rpart_4.1-9         
## [43] RSQLite_1.0.0        scales_0.2.4         sendmailR_1.2-1     
## [46] splines_3.1.3        stringr_0.6.2        survival_2.38-1     
## [49] tools_3.1.3          XML_3.98-1.1         xtable_1.7-4        
## [52] XVector_0.6.0
{% endhighlight %}
