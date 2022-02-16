---
layout: post
comments: true
title: 'virtualenvs or: How I Learned to Stop Worrying and Love not having sudo powers
  in a server'
date: 2015-01-28 07:45:06.000000000 +01:00
categories: []
tags: []
status: publish
type: post
published: true
---

Well, you might think, there is always `pip install --user`. Aha! Yes, that works most of the time, but not when some package upgrade (distribute for deepTools) conflicts with globally installed packages.

## The solution (for python packages)

_Python virtualenvs_ to the rescue!

These have been around for sometime, but I never felt inclined to dig into it until now (I can also be a bit of a _Luddite_). Put simply, _virtualenvs_ are reservoirs of custom python and package installations. In practice, one creates a folder where all the packages can be installed, and then simply source it. From that point on, everythying done in python is using whathever is in that folder or _virtualenv_. Even a copy of the python binary will be there.

One of the major appeals, for me at least, is that no _sudo_ is required for the set-up of the _virtualenv_ or installation of packages. Also, one can have multiple virtualenvs, say for different projects.

For a proper (practical tutorial) see this post by [Jamie Mathews][0].

In the meantime, _deepTools_ just finished compiling and I can now go to work.

[0]: http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/
