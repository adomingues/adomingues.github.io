---
layout: post
title: ChIP-seq coverage analysis
date: 2013-06-02 16:50:39.000000000 +02:00
categories: []
tags:
- bedtools
- Chip-seq
- genomics
- R
status: publish
type: post
published: true
---

#######
UPDATE
#######

It has been a long time since I wrote this post, and in between a wonderfull set of tools for coverage calculation and visualization has been published: [deepTools][0] does metagene analysis, and much, much more in a better than I could possibly do with my näive scripts.

Try it. It is worth it.
#######

There a few analysis tasks for which one would think a solution, or an easy to follow example, is readily available in the web (it has been a long time since the Web became the Internet). One such example is how to do coverage analysis of ChIP-seq, or any other data for that matter. That is, how are the reads distributed around a region of interest, commonly the TSS?

[HTSeq][1], the python library for NGS data analysis offers [that][2], but I prefer to do my plots in R and ggplot. I also find [bioconductor][3] nice but some of the objects and the packages literature hard to understand for someone looking for imminently practical information.  So I decided to ask around in my institute and create my own script. I like to do scripts that can be easily modified and also that generate plots that can be used with little or now change for presentations.

**Create the bed file with TSSs:**

Firstly, how to create the BED file with the TSSs using a combination of bioconductor and [bedtools][4] - it is quite fast and it very-well documented. This only needs to be ran once and there is a myriad of ways to do it. Of course one could also use any bed file with features of interest.

{% highlight R %}

######################  
### Load libraries ###  
library(GenomicRanges)  
library(GenomicFeatures)  
library(plyr)  
library(reshape)  
library(data.table)

### custom functions ##  
# string concatenation without space gaps  
concat <- function(...) paste(..., sep="")

################################  
## create TSS bed file (UCSC) ##  
# create a database of transcripts  
hg19RefGenes <- makeTranscriptDbFromUCSC(genome = "hg19", tablename = "refGene") # run only once  
hg19RefGenes # summary of the object  
saveDb(hg19RefGenes, file='~/Homo\_sapiens/hg19\_RefSeq\_genes.sqlite')  
# by saving the database one keeps a record of which version of the genome was used for the analysis. Very important for reproducibility.  
hg19RefGenes <- loadDb(file="~/Homo\_sapiens/hg19\_RefSeq\_genes.sqlite") # not necessary of useful if the the script need to be ran again later

# create an object with a list of transcripts grouped by gene  
transcripts <- transcriptsBy(hg19RefGenes, by="gene")

# convert to data frame:  
gene2tx <- as.data.frame(transcripts)

# Note that transcription\_start is always smaller than transcription\_end, even when the transcript is on the "−" strand. Hence, we have to use either the start or the end coordinate of the transcript, depending on the strand, to get the actual transcription start sites, i.e., the 5' ends of the transcripts:  
#http://www.bioconductor.org/help/course-materials/2009/EMBLJune09/Practicals/TSS/TSS\_plot.pdf  
tss <- ifelse(gene2tx$strand == "+", gene2tx$start, gene2tx$end)

tss.bed <- data.frame(seqnames = gene2tx$seqnames, start = tss, end= tss, strand=gene2tx$strand, transcriptID=gene2tx$tx\_name, type=rep("tss", length(tss)))

## just keep proper chromosomes  
tss.bed <- subset(tss.bed, str\_detect(seqnames, "^[[:alnum:]]\*$"))  
with(tss.bed, as.data.frame(table(seqnames)))    # list the frequency of genes per chromosome

# save bed file  
write.table(tss.bed, file = "~/Homo\_sapiens/hg19RefSeqTSS.bed", row.names = FALSE,  
col.names = FALSE, quote = FALSE, sep="\\t")

## use bedtools to increase the region around the tss  
system('  
chromInfo=~/Homo\_sapiens/UCSC/hg19/Annotation/Genes/ChromInfo.txt

bedtools slop -i ~/Homo\_sapiens/hg19RefSeqTSS.bed -g $chromInfo -b 1000 \> ~/Homo\_sapiens/hg19RefSeqTSS\_1000kb.bed  
')  
{% endhighlight %}

**Counting the number of reads per nucleotide**

Because I run over several bam files and I like to keep the file names tidy I've wrapped the bedtools function in a shell function so that the several analysis can ran in parallel to save time. Because this step takes a of time, it sends me an email at the end. This way I can do something else without being constantly looking at it. Hint: running it in  a [screen session][5] also saves a lot of headaches. If you are doing for a single bam file, or are not versed in shell scripting all you need is to run this:

{% highlight bash %}  
coverageBed -d -abam bamFile -b geneModel \> name.cov.bed  
{% endhighlight %}

But in all likelihood you will have several bam files:

{% highlight bash %}  
#################################################################################  
# This little shell script calculates coverage per base at any given annotation #

# wrapper shell function that does the coverage calculations and returns the files with nice names  
coverageAtFeatures(){  
# the input is a bam file that comes from sdout (see bellow)  
bamFile=$1  
name=$(basename ${bamFile%%.bam}) # to keep the directory tidy I rename the results files using the original bam file  
geneModel=$2 # also from sdout  
outDir=results/coverage # defines the output directory  
coverageBed -d -abam $bamFile -b $geneModel \> $outDir/$name.cov.bed # actually performs the coverage calculation.  
# note the -d, very important!

}  
export -f coverageAtFeatures # this add the function to the shell to be used

# Features:  
promoters=~/Homo\_sapiens/hg19RefSeqTSS\_1000kb.bed  
# bam folder:  
bam\_folder=~/chip\_seq/data/reads/

# now we can run the function defined above:  
# ls will list all the bam files in the folder and pass them to parallel.  
#Each bam file will the first argument of coverageAtFeatures  
# the second argument will be the bed file with the genomic regions of interest.  
# Parallel will run the analysis for 4 bam simultaneously - this # can vary depending on your computer.  
parallel  -i -j 4 bash -c "coverageAtFeatures {} $promoters" --  $(ls $bam\_folder/\*bam)"

# when all is done a message will arrive in your inbox.  
echo "Subject:Coverage done" |  sendmail -v myname@mymail.com \> /dev/null  
{% endhighlight %}

At the end of this we will have a file that contains something like:

{% highlight bash %}  
head chip\_experiment.cov.bed  
chr1 19922471 19924471 + NM\_001204088 tss 1 1  
chr1 19922471 19924471 + NM\_001204088 tss 2 1  
chr1 19922471 19924471 + NM\_001204088 tss 3 1  
chr1 19922471 19924471 + NM\_001204088 tss 4 1  
chr1 19922471 19924471 + NM\_001204088 tss 5 1  
chr1 19922471 19924471 + NM\_001204088 tss 6 0  
chr1 19922471 19924471 + NM\_001204088 tss 7 0  
chr1 19922471 19924471 + NM\_001204088 tss 8 0  
chr1 19922471 19924471 + NM\_001204088 tss 9 0  
chr1 19922471 19924471 + NM\_001204088 tss 10 0  
{% endhighlight %}

Col1-6 are the original entries of the bed file containing the TSSs, followed by the position of the base in the feature (remember this is a bp-resolution coverage) and the number of reads that mapped to the base.

Now we are ready calculate some statistics on own many reads map to each base surrounding the TSSs and plot these. In the next snippet I'll calculate the sum of reads per base but calculating the average requires a minor change to the script.  Since several of the operations take some time, I use the package doMC save time but this should work without it.

{% highlight R %}  
############################################  
## function to calculate coverage profile ##

## list coverage files  
path="./"  
cov.files <- list.files(path=path, pattern=".cov.bed")

## for multicore usage:  
library(doMC)  
registerDoMC(cores=10) # yay for our server

halfWindow=1000 # this half the size of the window around the TSSs

loadCovMat <- function(file){  
# read files  
cov.data <- read.table(concat(path, file), header=FALSE, sep = "\\t", stringsAsFactors = FALSE)

#     http://stackoverflow.com/questions/1727772/quickly-reading-very-large-tables-as-dataframes-in-r  
# cov.data <- fread(file)    # not used because strand not recognized  
names(cov.data) <- c("chr", "start", "end", "strand", "ID", "type", "position", "count")

# create a new ID to avoid repeated gene names - common issue with UCSC annotations  
cov.data.n <- transform(cov.data, newID = paste(ID, start, sep="\_"))  
# add base position with reference to the TSS.  
# very important: not the strand operation.  
cov.algned <- mutate(cov.data.n, position\_aligned=position-halfWindow,  
position\_str\_cor=ifelse(strand=="+", position\_aligned, -1\*position\_aligned))

# for the next operation R needs a lot of memory so I'll remove objs from memory  
rm("cov.data", "cov.data.n", "cov.data.t")

# counts the number of reads per position - the final result!  
res <- ddply(cov.algned, .(position\_str\_cor), summarize, sum=sum(count), .parallel=T)

## get condition name  
# important to have column with the name of the experiment when running multiple analysis  
sample <- strsplit(file, "\\\\.")[[1]][1]  
res$condition <- rep(sample, nrow(res))

return(res)  
}

## Run the function over all coverage files  
# the output is a list of data frames  
cov.l <- lapply(cov.files, loadCovMat)

## see the data  
lapply(cov.l, head)

## save the coverage data  
lapply(cov.l, function(x){  
write.table(x,  
concat(unique(x$condition), ".sum.cov"),  
row.names = FALSE,  
quote = FALSE, sep="\\t")})

# join the data frame in he list in one big data frame for plotting  
cov <- do.call("rbind", cov.l)

## 2 different plots:  
p.cov.all <- ggplot(data=cov, aes(x=position\_str\_cor, y=sum, color=condition)) + geom\_line()  
ggsave(filename = "./plots/coverage\_tss\_all.pdf", p.cov.all)

p.cov.wrap <- p.cov.all + facet\_wrap(~condition)  
ggsave(filename = "./plots/coverage\_tss\_wrap.pdf", p.cov.wrap)

{% endhighlight %}

If all went well this is the final result:  
[Plot 1][6]

[Plot2][7]

Al there is lacking now is putting all of this in a nice wrapper, which could be easily done, but I preferred to show the different stages of the process separated because it is easier to explain. I hope it helps.

[0]: https://github.com/fidelram/deepTools/wiki
[1]: http://www-huber.embl.de/users/anders/HTSeq/doc/overview.html "HTSeq overview"
[2]: http://www-huber.embl.de/users/anders/HTSeq/doc/tss.html#tss "HTSeq - TSS analysis"
[3]: http://www.bioconductor.org/help/course-materials/2009/EMBLJune09/Practicals/TSS/TSS_plot.pdf "TSS - An exercise with coverage vectors"
[4]: http://code.google.com/p/bedtools/
[5]: http://www.rackaid.com/resources/linux-screen-tutorial-and-how-to/ "Intro  to screen sessions"
[6]: http://movingtothedarkside.files.wordpress.com/2013/06/coverage_tss_all_mod.pdf
[7]: http://movingtothedarkside.files.wordpress.com/2013/06/coverage_tss_wrap_mod.pdf
