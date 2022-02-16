---
layout: post
title: Kill all jobs with a name
date: 2016-04-07 13:47:02.000000000 +02:00
categories: []
tags:
- queue
status: publish
type: post
published: true
---

I had some jobs waiting in queue with non-consecutive job id numbers, but all with the same job name. Mistakes were made and they needed killing. A solution would be to copy-paste all the relevant job IDs and go:

```bash
bkill 1001 1002 1030
```

But where is the fun in that? (Not mention that mistake could have been made and unintended jobs might have been canceled). 


# Solution

How to kill all jobs in queue with a certain name:

```bash
j=`bjobs | grep bam2bw | awk '{print $1}'`
bkill $j
```

`grep` will capture the jobs with a particular name, also using patters if needed, and `awk` will print the column with the job ids. `awk` could have also been used to capture the relevant jobs, but I feel more comfortable using `grep`.
