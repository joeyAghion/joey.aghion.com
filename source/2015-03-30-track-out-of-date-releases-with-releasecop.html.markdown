---
title: Track out-of-date releases with Releasecop
date: 2015-03-30 10:01 -04:00
tags: programming, heroku, git, github
---

Artsy practices a sort of [continuous delivery](http://en.wikipedia.org/wiki/Continuous_delivery): builds are automated, and there is always a deployable version of the codebase. Thus commits form a pipeline:

* From developers' working branches
* To the master branch
* Through a \[hopefully successful\] build
* Often to a staging environment
* Upon deploy, to the production environment

We have a number of tools to help us shepherd commits through this pipeline, but now that the number of apps and services we deploy has grown, sometimes things fall through the cracks. I just released a quick-and-dirty tool to help track how out-of-date different environments are: [**Releasecop**](https://github.com/joeyAghion/releasecop).

Just list the different environments for each project (by git repo or branch name) in a _manifest_ file, and the `releasecop check` command reports which commits have yet to be deployed to each environment. We set up a nightly Jenkins task to email us the results.
