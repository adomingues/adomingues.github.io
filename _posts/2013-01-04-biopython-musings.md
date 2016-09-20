---
layout: post
title: BioPython musings
date: 2013-01-04 16:23:34.000000000 +01:00
categories: []
tags:
- BioPython
- Course
- HTSeq
- NGS
- Python
- R
status: publish
type: post
published: true
---

On my quest to improve my scripting skils I have just recently taken the Introduction to [BioPython][0] course at the VIB in Leuven.

The course

It was all very well organized and the Instructor, [Kristian Rother ][1], was well prepared and engaging. Perhaps he was not expecting so many of wanting to learn more about handing NGS data but when he saw the interest, wrote overnight a custom script to parse and extract information SAM files. Not only it is useful, it also serves a practical example of several Python functions:

{% highlight python %}  
#!/usr/bin/env python

# TASK: find out how many sequences there are in the filtered dataset  
#       (the one printed at the bottom)

data = []

for line in open('example.sam'):  
if not line.startswith('$') and not line.startswith('@'):  
line = line.strip().split()  
record = {  
'QNAME' : line[0],  
'FLAG'  : int(line[1]),  
'RNAME' : line[2],  
'POS'   : int(line[3]),  
'MAPQ'  : int(line[4]),  
'CIGAR' : line[5],  
'RNEXT' : line[6],  
'PNEXT' : int(line[7]),  
'TLEN'  : int(line[8]),  
'SEQ'   : line[9],  
'QUAL'  : line[10],  
'optional' : []  
}  
for optional in line[11:]:  
record['optional'].append(optional.split(':'))  
data.append(record)

# filter all records between position 109200 and 110000  
count = 0  
for rec in data:  
if rec['POS'] > 109200 and rec['POS'] + len(rec['SEQ']) <= 110000:  
print rec['POS'], rec['SEQ']  
count = count + 1  
print "Total number of reads is: %i" % (len(data))  
print "Filtered reads is: %i" % (count)

{% endhighlight %}

Although I've learned a lot there were a problem that delayed the class progression: despite the organizers warning that the course was only for people that already knew Python basics, most of the class had not worked with python and a few had never programmed or even used command line to create a directory. This is not the organizer's or the instructor's fault and only delayed us in the first day. Actually those of us who already knew a bit of programming carried on with the exercises on our own.

**BioPython**

First let get out of the way that after the course ended, and for my own purposes - mostly analysis of NGS data - I did not see what could be done with BioPython that I could not already do with a combination of R and shell scripting. That said, parsing of files, for instance FASTA and even SAM appears to be faster and more intuitive than using R/Bioconductor packages. Parsing of files in Python appears to be much much simpler than in R. Of course there is a reason for that: R is for statistical analysis and was not developed to deal with strings. On the other hand Python has plenty of neat built-in functions to deal with strings that makes it much more powerful to deal with DNA/RNA/Protein sequences. BioPhyton seems to add even greater power for this.

Another point in favour of BioPhyton is its documentation which appears to be of a better standard than that of the average Bioconductor package. I've also quite liked the possibility of wrapping any variable in help() to check with function can be used with that particular variable - simple and affective.

Plotting is ok in Python with Matplotlib but its aesthetics are not even close to those of ggplot in R.

**NGS data in BioPython**

Before doing the course I did some search on what was available for the analysis of NGS data using Python and got the impression that Python programmers appeared to have disregarded this technology. There were plenty of scripts in Awk, Perl, etc to parse, manipulate and analyse SAM/BAM/FastQ files, but not in Python. I was wrong. During the course I've found out that BioPhtyton has [HTSeq][2]. I have yet to explore it completely bu tit looks like it could be a nice tool for quick QC and count reads over features - something I am yet to master in R.

All in all I think I will be using Python a lot more  in the future but still doing most of analysis with R and shell scripting.

[0]: http://www.bits.vib.be/index.php/training/92-python "BioPython course"
[1]: http://academis.sites.djangoeurope.com/ "Instructor"
[2]: http://www-huber.embl.de/users/anders/HTSeq/doc/index.html "HTSeq"
