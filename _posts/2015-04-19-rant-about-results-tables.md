---
layout: post
title: Table of results embed in a PDF
date: 2015-04-18 10:47:02.000000000 +02:00
categories: []
tags:
- Jekyll
status: publish
type: post
published: true
---


**Warning**: This is a rant.

I had to run a quick analysis on someone else's results. You probably know the drill: there is a paper which reported a list of differential expressed genes for a condition that is relevant to your project. One thinks:
> Great! Assuming the analysis is correct, I can poke around their data and compare/extract results that are relevant to my project.

That is where the fun ends. In this particular paper, they where kind enough to report the processed tables of results in a single PDF.

Seriously? All 4 tables of results were embedded in a single PDF. The lists of up and down-regulated genes spanned several pages. Is it that hard to have supplement text files with the relevant tables? Or if that is not convenient for data presentation (yes, there were highlighted row in the tables), at the very least make the original excel tables available. Unless all the calculations were painstakingly done by hand and noted down in Word, it seems that it was more difficult to move the table from Excel -> Word -> PDF than just published the excel files in the first place.


## Extraction of table from PDF to R-friendly format
To extract the list of differentially expressed genes embedded in the supp. data pdf, I used okular following this [tip](http://stackoverflow.com/a/11437638/1274242). However, when copying directly to Oo Calc I stumbled upon some encoding problems, so in the end table was copied to a text editor (the column that was relevant to me). It is not ideal but solved the problem. It took me an hour all in all.


Of course, I could have download the raw reads, QC, map, count, re-do the analysis, compare to the published results to make sure all is kosher, but having the table of DEG saves a lot time that we can focus on extracting biologically relevant insights. That is what science should be about and sharing knowledge, in the format of papers, should be a help in this endeavor and not an hindrance.
