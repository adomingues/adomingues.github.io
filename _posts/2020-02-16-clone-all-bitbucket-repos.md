---
layout: post
title: Clone all repositories from a user (bitbucket) - API2.0
date: 2020-02-18 21:52:15.000000000 +01:00
categories: []
tags:
- bash
- bitbucket
- mercurial
status: publish
type: post
published: true
---

My [post](http://adomingues.github.io/2015/01/10/clone-all-repositories-from-a-user-bitbucket/)  about cloning all bitbucket, originally [posted](https://movingtothedarkside.wordpress.com/2015/01/10/clone-all-repositories-from-a-user-bitbucket/) on my old wordpress blog, got some attention over the internet. Well, three or four mentions. Sadly the information in there has been outdated due to Bitbucket's [API changes](https://developer.atlassian.com/cloud/bitbucket/deprecation-notice-v1-apis/). That added to the shocking decision by Bitbucket to stop [supporting mercurial repos](https://bitbucket.org/blog/sunsetting-mercurial-support-in-bitbucket) and delete them (!) pushed me to finally download all my repos (again) and eventually move to gitlab. Maybe in a follow up post I will detail how I converted all hg repos to git. 

> June 1, 2020: users will not be able to use Mercurial features in Bitbucket or via its API and all Mercurial repositories will be removed.
> 
## Donwload data for all repositories

The first step is to download as much information as possible to parse from that the download links.
 
__WARNING__: if you have more than 100 repositories keep reading.

```bash
 USER=username
 PASS=password

# source: https://stackoverflow.com/questions/40429610/bitbucket-clone-all-team-repositories
curl --user $USER:$PASS https://api.bitbucket.org/2.0/repositories/$USER?pagelen=100 > repoinfo.json
```

## Get all the links

With this information at hand we can now extract the ssh link needed to clone the repo. If you only want to clone git repositories, then [this](https://gist.github.com/eeichinger/c2eb46fbe7d5e2f49eba3bfaf5471759) or any [these](https://stackoverflow.com/questions/40429610/bitbucket-clone-all-team-repositories) solutions will do the trick. However for my use case there are a couple of complications:

1. repositories are version controlled with either git or hg (that's how I started);
2. repositories might have already been cloned in the destination directory;
3. I have more than 100 repos but the API limits the call to 100 results per page

To address issue number 1, [this solution](https://stackoverflow.com/a/58752204) would be enough, but it doesn't help for #2. So using that answer as starter code, I modified it pull & merge uncommitted changes to previously downloaded repos. This works for me because I am the single developer of those projects and because I only need to donwload the repos in it's current version as a sort of backup. If this not necessary simply removed the `if [ -d "$repo" ]` test.

Solving #3 is a bit more complicated, but thankfully someone else came up with a [solution](https://stackoverflow.com/a/51142042). As an aside, this is an example of beauty of making things out in the open: my original post served as an inspiration for this answer, and I am now using it to solve a new problem. We've gone full circle. 

Now for the code:

```bash
cwd=`pwd`
 USER=username
 PASS=password # space to void this getting into history. If there are better options to anonymize the password I would like to know

# source: https://stackoverflow.com/a/51142042
for i in {1..2}; do
	curl --user $USER:$PASS "https://api.bitbucket.org/2.0/repositories/$USER?pagelen=100&page=${i}" > repoinfo.page${i}.json
done

## [0].href will return the https link and [1].href the ssh link
cat repoinfo.*.json | jq -r '.values[] | .links.clone[1].href' > repos.txt

# source: https://stackoverflow.com/questions/40429610/bitbucket-clone-all-team-repositories
for repo in `cat repos.txt`; do
	projectname=`basename ${repo}`
	if [[ $repo == *".git" ]]; then
		echo "It's a git!"
		command="git"
	else
		echo "It's a mercurial!"
		command="hg"
	fi

	if [ -d "$projectname" ]; then
		echo "Project directory exists. $repo will be updated"
		cd $cwd/$projectname
		$command pull
		cd $cwd
	else
		echo "Cloning" $repo
		$command clone $repo
	fi
done
```

It is possible to refine this code by directly retrieving the information about the type of repo, hg or git, but I don't have the time to fix what is good enough for my purposes. If anyone wants to go in that direction, this how one can extract that information:

```bash
curl --user $USER:$PASS https://api.bitbucket.org/2.0/repositories/$USER?pagelen=100 | jq -r '.values[]  | {scm, links:.links.clone[0].href}'
```

__Pro tip__
There is a very good website to test interactively jq queries to parse json: [website](https://jqplay.org/). Very useful. 