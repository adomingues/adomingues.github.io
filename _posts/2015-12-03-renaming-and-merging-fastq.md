---
layout: post
title: Merge fastq sample from different lanes and rename them
date: 2015-12-03
categories: []
tags:
- bash
- fastq
status: publish
type: post
published: true
---


This is something I need to do often and a collegue asked me how to do it herself. So the best way to share is to post it in the blog.


# Problem

Quite often samples are sequenced in multiple lanes, or for whatever reason are split in multiple file, which we receive. Before processing, I like to join the fastq files and rename the at that point - afterwards everything  will have plot names which are mostly presentation/publication ready, so one last thing to change manually. 


# 1st step: merge

```bash
PROJECT='my_project'

# create project folder
mkdir -p ${PROJECT}/data/reads/03_December_2015
cd ${PROJECT}/data/reads/03_December_2015

# copy original reads - these will always be kept!
rsync -r -t -x -v --progress -u -l -z -s /fsimb/exchange/imb-genomicscf/AG_Ketting/imb_ketting_2015_22/ ./

# merge the reads in parallel taking advantage of the cluster
mkdir -p logs
for f in $(find ./ -type d -name "*imb*");
do
    echo $f
    bsub -q short -n 1 -app Reserve3G -o logs/output.txt -J logs/mergeFastQ_"$f" -e logs/mergeFastQ_"$f".err.log mergeFastQ.sh $f "single"
done

```

This will call `mergeFastQ.sh` that changes to each sample directory and merges the files therein (save it in a folder that is on your `$PATH`):

```bash
#!/usr/bin/env bash

# mergeFastQC.sh
# António Domingues
# 22.01.2015

cd $1
# outname=`echo $1 | sed -r 's/_[a-Z]{6}_.*(_R[1-2]).*$/\1/'`
outname=${PWD##*/}
echo $outname
# read1=`find -name "*R1*.fastq.gz"`
# read2=`find -name "*R2*.fastq.gz"`


if [ "$2" = "paired" ]; then
   echo "Paired"

   cat *_R1_*.fastq.gz > "$outname"_R1.fastq.gz

   cat *_R2_*.fastq.gz > "$outname"_R2.fastq.gz

elif [ "$2" = "single" ]; then
   echo "Single"

   cat *_R1_*.fastq.gz > "$outname".fastq.gz

fi
```


# 2nd step: rename

Once the merge is confirmed, merged files were renamed and moved to a merge folder. Original files were compressed in a folder.

```shell

mkdir -p pooled

for f in $(find ./ -type f -name "Sample*.fastq.gz");
do
    n="pooled/"$(echo `basename $f` | sed -e 's/Sample_imb_ketting_2015_22_[0-9]*_//')
    mv $f $n
done

# compress the original reads in tar and cleans up
tar cf Sample_imb_ketting_2015_20.tar $(find ./ -type d -name "*imb*")
rm -rf $(find ./ -type d -name "*imb*")
```


# 3rd step: processing

Once we have this we start the removal of poor quality reads, fastQC, mapping, etc. For his I take advantage of the Institutes's pipelines, [NGSpipe2go](https://github.com/imbforge/NGSpipe2go), to which I added a small RNA-seq pipeline. You can download the whole thing [here](https://github.com/adomingues/NGSpipe2go) to your `${PROJECT}` folder and uncompress the zip file. Then to run you will only need one command: `bpipe run smallrnaseq_v0.1.txt data/reads/03_December_2015/*.fastq.gz`. All the required scripts and files should be accessible.
