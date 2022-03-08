---
layout: post
title: Convert multiple mercurial repos to git
date: 2021-03-08 22:17 +0100
categories: []
tags:
- bash
- git
- mercurial
- gitlab
status: publish
type: post
published: true
---

For historial reasons many of my I have dozens of version-controlled projects using mercurial (hg) hosted on Bitbucket. Since Bitbucket not only discontinued support for mercurial but also decided to [delete those repositories](https://bitbucket.org/blog/sunsetting-mercurial-support-in-bitbucket), I have been planning to convert those to git - it only took me two years. Since I was getting my hands dirty with this, I took the opportunity to convert hg to git and upload to gitlab in one fell swoop. Here is how.  

Luckily there is an off the shelf tool to help with the process, `fast-export` and a tutorial:

https://git-scm.com/book/en/v2/Git-and-Other-Systems-Migrating-to-Git

After testing the conversion in a project, most of the work was to automatize the process as much as possible using bash loops. 

The first step was to find all mercurial repos, and then get all my mess of user ids and harmonize them. 

```bash
cd ~/sandbox

git clone https://github.com/frej/fast-export.git

# find all mercurial projects
all_projs=$(find  ~/Documents/projects -name ".hg" -type d -exec bash -c "echo "{}" | sed 's/.hg//g'" \;)
echo "${all_projs}"

## gets users
for proj in $all_projs; do
    echo $proj
    cd $proj
    hg log | grep adomingues
    hg log | grep user: | sort | uniq | sed 's/user: *//' >> ~/sandbox/gh_users.tmp
done
cat ~/sandbox/gh_users.tmp | sort | uniq | sed -E 's/(.*)/"\1"="António Domingues <amjdomingues@gmail.com>"/g' > ~/sandbox/gh_users.txt
rm ~/sandbox/gh_users.tmp

cat ~/sandbox/gh_users.txt 
```

>"adomingues"="António Domingues <amjdomingues@gmail.com>"
"amjdomingues"="António Domingues <amjdomingues@gmail.com>"
"Antonio Domingues <amjdomingues@gmail.com>"="António Domingues <amjdomingues@gmail.com>"

The file `gh_users.txt` is useful, though not required, for the conversion. 

If I was doing the conversion to git alone, the code below would have been simpler, but even with the added upload to gitlab it's not that complicated. for each `hg` folder:

- get the project name
- initiate a git mirror project on a separate folder
- convert to `.git`
- create a new project with that name in gitlab.com 
-rename `master` to `main` [optional]
- push the lot to gitlab

Don't forget to set-up ssh for gitlab and create an API token before doing this (and getting a gitlab account?)

```bash
gitlab_user="amjdomingues"

for proj in $all_projs; do
    proj_name=$(basename $proj)
    new_proj="${HOME}/sandbox/converted_to_git/${proj_name}"

    echo $new_proj
    git init $new_proj
    cd ${new_proj}
    ## convert to hg in a separate folder
    ${HOME}/sandbox/fast-export/hg-fast-export.sh -r $proj -A ${HOME}/sandbox/gh_users.txt
    git checkout HEAD ## https://github.com/frej/fast-export
    
    curl --data "name=${proj_name}" --header "PRIVATE-TOKEN: YOURTOKEN" "https://gitlab.com/api/v4/projects"

    git branch -m master main

    git remote add origin git@gitlab.com:amjdomingues/${proj_name}.git
    git push -u origin --all
    git push -u origin --tags
done

```

The tricky parts were:
- checkout all changes with `git checkout HEAD` which is not documented in the tutorial but mentioned in the README of [fast-export](https://github.com/frej/fast-export).
- figure out the right API call to create the gitlab project, documented [here](https://docs.gitlab.com/ee/development/documentation/restful_api_styleguide.html)

Missing accomplished!

There were few errors along the way, but when I checked it was all very innocuous. These are mostly data analysis projects that I don't intended to pick up anytime soon, but there are some snippets of code here and there atht I would like to consult on a regular basis. That said, I kept an offline backup of the projects just in case something went wrong.

**Bonus material**

It happens that some of bitbucket projects were already version-controlled with git, but due to my unhapiness with Bitbuctek's decision, I decided to move them all to gitlab. 

```bash
all_projs=$(find  ~/Documents/projects -name ".git" -type f -exec bash -c "echo "{}" | sed 's/.git//g'" \;)
echo "${all_projs}"

gitlab_user="amjdomingues"

for proj in $all_projs; do
    proj_name=$(basename $proj)
    cd ${proj}    
    echo "${proj_name}"

    curl --data "name=${proj_name}" --header "PRIVATE-TOKEN: YOURTOKEN" "https://gitlab.com/api/v4/projects"

    git branch -m master main

    git remote set-url origin git@gitlab.com:amjdomingues/${proj_name}.git
    git push -u origin --all
    git push -u origin --tags
done

```

The final step was to overwrite - after backing up - the `hg` repos with the `git` versions.

```bash
## tidy up
rsync -r -t -v --progress -u -s ${HOME}/sandbox/converted_to_git/ ${HOME}/Documents/projects/
```
