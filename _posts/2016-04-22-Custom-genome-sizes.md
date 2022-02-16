---
layout: post
title: Custom chromosome sizes for pybedtools
date: 2016-04-22 16:15:02.000000000 +02:00
categories: []
tags:
- pybetools
- pysam
- python
- genome
status: publish
type: post
published: true
---


I use `pybetools` a lot in my little script. One issue that I have encountered recentely was when using those scripts with a custom genome, in this case mapping to the transcriptome. One of my scripts calculates using `genome_coverage(bg=True, genome=genome)` and the argument `genome`, is an input from the command line. 

And what is this `genome` I speak of? It is a table that informs `pybedtools` of the sizes of the chromosomes in the reference. Useful among other things to calculate coverage and to avoid extending genomic regions more that the size of the reference chromosome with `slop`. 


# What is the problem?

Having to create custom tables for each genome might be annoying. Also, I would like to reduce the number of arguments in my scripts. So while this is not an issue for most application, at some point one will run into annoyances that are part and parcel of custom genomes/annotations/applications.


# The solution

There is this thing called `samtools idxstats` which outputs the chromosome lengths (and read counts per chromosome) for bam files. It just happens that the main input of my scripts tend to be bam files, and that `idxstats` has been implemented in `pysam`. This means that with a simple function the chromosome lengths could be derived from the bam without extra hassle (said he before the bugs crept in).

```python
def get_chrom_lengths(path_to_bam):
   '''
   Uses pysam to retrieve chromosome sizes form bam.
   Useful helper to use with some pybedtools functions (e.g. coverage), when a bam was mapped with custom genome not available in UCSC.
   Input: path to bam file (should be indexed)
   Output: dictionary.
   Example ouput:
   {'chr4': (0, 1351857), 'chr3L': (0, 24543557), 'chr2L': (0, 23011544), '*': (0, 0), 'chrX': (0, 22422827), 'chr2R': (0, 21146708), 'chr3R': (0, 27905053)}
   '''
   import pysam
   idx = pysam.idxstats(path_to_bam).splitlines()
   chromsizes = {}
   for element in idx:
      stats = element.split("\t")
      chromsizes[stats[0]] = (0, int(stats[1]))
   return chromsizes
```

and a little test will show that the chromosome sizes obtained with this function are equivalent to those retrieved by `pybedtools` from UCSC (or their database):

```python
import pybedtools
bam = pybedtools.example_filename('x.bam')
pysam.index(bam) # with indexing it will not work

#my function
chromsizes = get_chrom_lengths(bam)

# pybedtools in-built function
dm3 = pybedtools.genome_registry.dm3.euchromatic
print dm3

# test
a = pybedtools.example_bedtool('x.bed')
cov_dm3 = a.genome_coverage(bg=True, genome='dm3')
cov_chrsizes = a.genome_coverage(bg=True, genome=chromsizes)

print cov_dm3.head()
print cov_chrsizes.head()

cov_dm3 == cov_chrsizes
```


Note: it is working with pysam 0.9.0 but might untested for other versions where the parsing might be [different](https://github.com/pysam-developers/pysam/issues/245).
