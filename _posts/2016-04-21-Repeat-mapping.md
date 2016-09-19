---
layout: post
title: Repeat mapping
date: 2016-04-21 13:15:02.000000000 +02:00
categories: []
tags:
- Mapping
- Repeats
- Review
- small RNA-seq
status: publish
type: post
published: true
---

Most of the projects I am involved with deal with mapping reads to repeat regions of the genome. Specifically transposons. While not all genomic repeats have **exactly** the same sequence, it is nonetheless challenging to accurately map as many reads as possible - more reads mapped -> more information (for the same €€).


# What is the problem?

The issue is what to with reads whose mapping location is uncertain, or rather that map to more than one genomic position. What to do next? We can overall find in the literature two consensual ways to deal with this problem (at the level of mapping):
- retain only reads that map one (unique mappers) gives us confidence about the read position in exchange for loss of information;
- randomly assign to a single location a read that map multiple times (multi-mappers), ensuring increased read numbers at the expense of uncertainty about the origin of the reads.

There also post-mapping strategies to quantify expression of repeat regions. For instance one can allow random assignment of multimapping reads and simply quantify classes of Repeats. The assumption here is that if a read maps to multiple times with the same confidence it will probably be to different copies of the *same* element. Exact genomic location is not important, only the element. ERANGE on the other hand, uses uniquely mapping reads to calculate which repeats (genes) are more likely to be expressed, and by how much, and then assigns the multimapping reads to genes based on the estimate.

Regardless, there will genes with so many copies with almost identical sequences in the genome, that it will be virtually impossible to map reads (or quantify expression) to any of those copies. This is the *black hole*.


# Beware of Dragons

The above introduction serves to say that when it comes to the mapping in repeat sequences/regions, there is no right way of doing it, and compromises will have to be taken. These choices should always be present when analyzing and interpreting the downstream results.

So, there is no *right* solution for this problem, but could there be a (or many) wrong way(s) of doing it?

There are certainly two papers that I am aware of that came under scrutiny for the strategy used when dealing with reads mapping to repetitive regions. The most recent one, [Samans et al](http://www.ncbi.nlm.nih.gov/pubmed/24998597), was dissected [here](http://www.ncbi.nlm.nih.gov/pubmed/27046835). I will not review in detail the arguments, but the main contention is that by using the bowtie parameter `-a` Saman et al. is counting each multimapping read multiple times. Basically if a read maps to 2 positions in the genome, it will be assigned to both positions. 

1 becomes 2.

2 can be hundreds.

As we can expect, Royo et al. when re-analysing the data shows that this affects disproportionately reads mapping to repeat regions, with a large increase in mappings to these regions. This is significant because Saman et al. found a strong enrichment of nucleosomes at repetitive regions. Two senior authors of the Samans et al study replied and [stood by their conclusions](http://www.ncbi.nlm.nih.gov/pubmed/27046829) arguing that these dark regions of the genome (more than half of the mammalian genome) must be considered in the data analysis and not simply throw away the data. This is very true and I agree with their sentiment, but facing the choice between ignoring reads for which the location I am unsure of, or ignoring them, I will stray on the side of caution. I will definitely avoid multiple assignment (double counting) of reads.

Previously, a paper by [Huang et al](http://www.ncbi.nlm.nih.gov/pubmed/23434410) found enrichment of Piwi (an Argonaut protein involved in transposon repression) in transposon loci, using ChIP-seq. [Marinov et al](http://www.ncbi.nlm.nih.gov/pubmed/25805138) re-analyzed the data, and in my opinion the most important control - simply swapping input background with the Piwi IP for peak calling - showed similar enrichment at transposons. In a reply, [Lin et al.](http://www.ncbi.nlm.nih.gov/pubmed/25805139) upon re-analysis of their own data, agree that the genomic targets of Piwi are still unknown. Once again the results falter due (mostly) to the handling of multimapping reads. There are other issues, namely the non-standard peak-calling, which should be an eye opener to anyone interested in mapping to transposons/repeats.

I did not dwell in the biological questions being address by Samans et al. or Huang et al., nor on the finer details. The goal is to bring some attention, and focus my mind in the issue of multimapping reads. I live in constant paranoia that a simple mistake (switching `-M` for `-m` in bowtie), or lack of understanding of the best practices will result in poor results. It can happen to anyone, so please go ahead and read the papers, specially the re-analyses and rebuttals. It is just a few pages each and it save many headaches in the future.


# Is there a way to kill the Dragons?

As you can probably notice, this multimapping reads issue is a lot on my mind. Sometime ago after lab discussions, and embryonic idea of how assign these reads started taking shape. Something like:

1. map reads and keep multi-mappers.
2. use uniquely map reads to estimate local coverage
3. assign multi-mappers to regions based on the coverage. That is, if there are 100 non-redundant reads that map with equal confidence (as assigned by the mapper) to two locations in the genome, one with coverage=10 and the other with coverage=1, the 9/10 of the 100 reads would go to a position 1 and 1/10 to position 2.

Note that this is not the same as existing solution of **counting** a read proportionally to the number of mapping positions. The output would be a sam/bam alignment file in which every single read would have a single genomic position. This alignment file could then be used for instance to call peaks.

With my very näive solution also came the though: "someone smarter than me already though to something like this". So I put this on my TODO when-there-is-time list, and kept an eye in the literature.

Now, a group of smarter people than me did put this solution into a proper package, [ShortStack](http://dx.doi.org/10.1101/044099):

> **Improved Placement of Multi-Mapping Small RNAs**
> Nathan R. Johnson, Jonathan M. Yeoh, Ceyda Coruh, and Michael J. Axtell
> Article Summary:
High-throughput sequencing of small RNAs (sRNA-seq) is a frequently used technique in the study of small RNAs. Alignment to a reference genome is a key step in processing sRNA-seq libraries, but suffers from enormous rates of multi-mapping reads. Current methods for sRNA- seq alignment either place these reads randomly or ignore them, both of which distort downstream analyses. Here, we describe a locality-based weighting approach to make better decisions of placement of multi-mapped sRNA-seq data, and test our implementation of this method. We find that our method gives superior performance in terms of placing multi-mapped sRNA-seq data. An implementation of our method is freely available within the ShortStack small RNA analysis program. Use of this method may dramatically improve genome-wide analyses of small RNAs.

Isn't great? They even put it up as preprint in biorxiv. This is quite crucial because in a typical small RNA-seq experiment we only get about 20-30% of unique mappers, and 50-60% of reads are multi-mappers. If the results of the paper hold true, about 70-30% of these multi-mappers would be "saved" by ShortStack. I still have to test it for ChIP-seq, but it looks promising. 

ShortStack is not the panacea for all the multimapping issues, and authors acknowledge that for some reads assigning a single genomic position, but it should increase the number of available reads for downstream analysis, post-mapping, without sacrificing accuracy. 

Good times ahead.
