---
layout: post
title: Calculating and plotting mapped reads - a simple R/shell script
date: 2012-10-25 09:47:02.000000000 +02:00
categories: []
tags:
- NGS
- QC
- R
status: publish
type: post
published: true
---

As this is my first post I'll start with something very simple I did recently. The problem is simple: "how many reads did does my deep-seq experiment have and how many were mapped to the genome?". I am now dealing with several RNA-seq and ChIP-seq experiments and having a pipeline to get this info fast for several files is important to me.

There are several suggestion out there on how to it, but I wanted something that could be used routinely for several files per experiment, and that could output the results in a visual manner (and also a table).

Using SAMtools as suggested [here][0] (thanks!) I've started with a simple bash script that would output the mapped reads into a text file and possible do the maths:

{% highlight bash %}  
#! /bin/bash

# file header  
echo "bam_file total mapped unmapped"

# loop to count reads in all files and add results to a table  
for bam_file in *.bam  
do  
total=$(samtools view -c $bam_file)  
mapped=$(samtools view -c -F 4 $bam_file)  
unmapped=$(samtools view -c -f 4 $bam_file)  
echo "$bam_file $total $mapped $unmapped"  
done  
# ultimately the data is saved in a space-separated file

# saved in a file called "CountMappedReads.sh"

{% endhighlight %}

Unfortunately arithmetic operations are not something advisable to do in shell - in my particular case because I am using a remote server without a necessary package. That minor setback led to think about doing in [R][1]: It can send commands to the shell; it is certainly built for mathematical/statistical operations and it has [ggplot][2]!

So this how the code looks like this:

{% highlight R %}  
# count reads using SAM tools  
library(ggplot2)  
library(reshape2)  
library(plyr)

# function to call BAMtools and store reads count in a text file  
CountReadsBAM <- function()  
{  
command= "~/bin/CountMappedReads.sh > MappedReads.txt"  
try(system(command))  
res=read.table("MappedReads.txt", header=T)  
return(res)  
}

# calling the funtion will output read count in an object:  
ReadCount <- CountReadsBAM()

# a bit of polishing to remove total (not needed for plotting)  
ReadCountSmall <- data.frame(BAM = ReadCount$bam_file, Mapped = ReadCount$mapped, Unmapped = ReadCount$unmapped)

# ggplot needs data in a specific layout  
MeltedReadCount = melt(ReadCountSmall, id=c('BAM'))  
names(MeltedReadCount) <- c('BAM', 'Mapping', 'Reads')

# calculate the fraction of mapped reads  
ReadsFraction <- ddply(  
MeltedReadCount,  
.(BAM),  
summarise,  
Count.Fraction = Reads / sum(Reads)  
)

# sort the data frame and add fraction to the data frame  
to_graph <- cbind(arrange(MeltedReadCount, BAM), fraction = ReadsFraction$Count.Fraction)

# Now all we have to do is plot the data  
gp <- ggplot(data=to_graph, aes(x=BAM, y=Reads, fill=Mapping)) +  
geom_bar(stat="identity",position = "stack") +  
geom_text(aes(label=paste(round(fraction*100),"%", sep="")),size = 3,vjust=0,position="stack") +  
opts(axis.text.x=theme_text(angle=90))

pdf("MappedReads.pdf")  
gp  
dev.off()  
{% endhighlight %}

To execute all you have to is to go to the folder that contains the .bam files and excute from the shell:

{% highlight bash %}  
Rscript ~/bin/CountMappedReads.r  
{% endhighlight %}

This is how the plot looks like:

[![](assets/mappedreads.png?w=300)][3]

This of course very simple, which also means very easy to change for ones specific needs, but serves the purpose of having quick visual information on how data mapped and perhaps present in group meetings. That said, never underestimate the power of visual information and one of the many beauties of ggplot is that the plots can be customized to exhaustion (I'll update this plot soonish to include the % mapping: UPDATED).

This simple plot contains a lot of useful information: (i) How many reads we collected; (ii) how many were mapped (the total numbers are visible in the Y-axis); (iii) visual information on the proportion of mapped reads; and just in case, (iv) also the actual proportion of mapped and unmapped reads. And this for 12 mapped files!

[0]: http://left.subtree.org/2012/04/13/counting-the-number-of-reads-in-a-bam-file/ "Counting mapped reads"
[1]: http://www.r-project.org/ "R"
[2]: http://ggplot2.org/ "ggplot2"
[3]: http://movingtothedarkside.files.wordpress.com/2012/10/mappedreads.png
