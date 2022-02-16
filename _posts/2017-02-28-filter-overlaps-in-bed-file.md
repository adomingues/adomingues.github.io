---
layout: post
title: Filter overlapping features in bed file
date: 2017-02-28 11:32:02.000000000 +02:00
categories: []
tags:
- bedtools
- bed
status: publish
type: post
published: true
---

I was doing something that should be easy but I couldn't find a solution online: remove overlapping coordinates in a single bed file and these must be within a certain distance. In this example:

```bash
chr1 10  20
chr1 50 60
chr1 25 35
```

I would like to remove genes that are "overlapping" within 10 bp, so the output would be: 

```bash
chr1 50 60
```

Furthermore, it should consider strandness. 

After playing around with many tools, I ended up cobbling together a one-liner. Some real-life data: 

```bash
cat repeat_masker.zv9.ucsc.noZv.bed | sortBed -i stdin | head -5
```

> chr1    37      120     En-Spm  ENSPM-6_DR      +       DNA
> 
> chr1    60      194     Helitron        Helitron-N3_DR  -       DNA
> 
> chr1    1100    1123    Low_complexity  AT_rich +       Low_complexity
> 
> chr1    1253    1302    Low_complexity  AT_rich +       Low_complexity
> 
> chr1    1472    1508    Low_complexity  AT_rich +       Low_complexity

And the solution:

```bash
cat repeat_masker.zv9.ucsc.noZv.bed | \
    sortBed -i stdin | \
    mergeBed -d 20 -i stdin -c 4,5,6,7 -o distinct | \
    grep -v ',' | \
    head -5
```

> chr1    1100    1123    Low_complexity  AT_rich +       Low_complexity
> 
> chr1    1253    1302    Low_complexity  AT_rich +       Low_complexity
> 
> chr1    1472    1508    Low_complexity  AT_rich +       Low_complexity
> 
> chr1    2851    3060    hAT     ANGEL   +       DNA
> 
> chr1    3686    3727    Simple_repeat   (TG)n   +       Simple_repeat
> 

Now the first two elements are now gone.

So what did is happening here? I figured that merging the overlapping features is the way to go, since it would “remove” the overlapping ones. Merging usually does not keep strand, so I had to to change the command accordingly. Note that the `-s` was not used, otherwise this would keep overlapping genes in opposite strands. The beauty of this, is that it will also concatenate the overlapped feature IDs, and they can excluded using that information. Simple.

The above still feels like too much of an hack, and it will not work with a GTF but will do for now. Maybe someone has a solution that I missed?