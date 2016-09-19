---
layout: post
title: Clone all repositories from a user (bitbucket)
date: 2015-01-10 13:12:15.000000000 +01:00
categories: []
tags:
- bash
- bitbucket
- mercurial
status: publish
type: post
published: true
---

Now that I have computer, the next step is to install everything I use on daily basis, and set-up my projects space. This involves cloning all my repositories. I could do this manually one by one, but what is the fun in that?

## Solution 1

My rule is that "if there is a way to make some task more efficient programatically, some one else thought of it before, so google it before starting scripting". So I did, and of course someone else came up with a [solution][0]:

{% highlight bash %}  
#!/bin/bash
#Script to get all repositories under a user from bitbucket
#Usage: getAllRepos.sh [username]
## credit to Harold Soh
 
curl -u ${1}  https://api.bitbucket.org/1.0/users/${1} > repoinfo
for repo_name in `grep \"name\" repoinfo | cut -f4 -d\"`
do
    hg clone ssh://hg@bitbucket.org/${1}/$repo_name
done
{% endhighlight %}

This however did not work perfectly for me. When downloading the rep info, the json config comes poorly formatted as a single line. This meant that the line `grep \"name\" repoinfo | cut -f4 -d\"` was return `scm`, which obviously is not what one of my repositories.

## Fix

So I hacked a little and came up with a not-very-elegant-but-working-solution to parse the json:

{% highlight bash %}  
#!/bin/bash
#Script to get all repositories under a user from bitbucket
#Usage: getAllRepos.sh [username]
#source: http://haroldsoh.com/2011/10/07/clone-all-repos-from-a-bitbucket-source/
 
curl -u ${1} https://api.bitbucket.org/1.0/users/${1} > repoinfo
# curl -u adomingues https://api.bitbucket.org/1.0/users/adomingues
# cat repoinfo
 
for repo_name in `cat repoinfo | sed -r 's/("name": )/\n\1/g' | sed -r 's/"name": "(.*)"/\1/' | sed -e 's/{//' | cut -f1 -d\" | tr '\n' ' '`
do
    echo "Cloning " $repo_name
    hg clone https://${1}@bitbucket.org/${1}/$repo_name
    echo "---"
done
{% endhighlight %}

This basically the same script as that of Harold, but with a more complex parsing:  
- `sed -r 's/("name": )/\n\1/g'` makes sure that the repo name is not at the start of each line;  
- `sed -r 's/"name": "(.*)"/\1/'`, removes the string "name" at the beginning of the line;  
- `sed -e 's/{//'` removes a funny curly bracktet ar the start of the file;  
- `cut -f1 -d\"`, separates and keeps the actual repository name;  
- `tr '\n' ' ' '` just removes the new line character creating a list of repository names to be looped.

_Et voilá_! There is probably some less convoluted away of going about it, probably involving regex, but in someone else's wise words: "if you need regex to solve a problem, you now have 2 problems".

The only minor inconvenience is that I need to input my password for each repository. There might be a solution for this, but I have not found it yet.

[0]: http://haroldsoh.com/2011/10/07/clone-all-repos-from-a-bitbucket-source/
