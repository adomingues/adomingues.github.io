---
layout: post
title: One liners, snippets and small scripts
date: 2012-11-26 10:32:22.000000000 +01:00
categories: []
tags:
- cut
- gff3
- grep
- snippet
status: publish
type: post
published: true
---

Often I use one liners or small scripts for useful task but I keep forgetting about those. So I'll just put them here for future reference.

Count number occurrences in a column/field. In this case, how many lines in a GFF3 file exist for each chromosome.

{% highlight bash %}  
cut -f1 hg19.GFF3 | sort | uniq  -c | sort  
{% endhighlight %}

This can be further refined and count only genes per chromosome using a simple grep:

{% highlight bash %}  
grep "gene" hg19.GFF3 | cut -f1 | sort | uniq  -c | sort  
{% endhighlight %}
