---
title: Using `git bisect run` with RVM
date: 2012-07-24
tags: git, programming, tools
---

If you're not familiar with `git bisect`, [check out the docs](http://www.kernel.org/pub/software/scm/git/docs/git-bisect.html). It's an invaluable tool for locating the commit that introduced a failure in your git repository (via binary search). You simply test and mark different code states as _good_ or _bad_, and `git bisect` narrows down the offending code.

If you can test whether a given commit is good or bad with a single command, it gets even better. The `git bisect run` command is autopilot for `git bisect`--just specify the command to test each commit and let it loose:

    git bisect run bundle exec rspec spec/models/whoops.rb

_BUT_, if you use [RVM](https://rvm.io/) and your environment is anything like mine, you might see errors implying that the command's `PATH` isn't set correctly. The reason is that `git bisect` prepends `/usr/libexec/git-core:/user/bin` to the `PATH` when executing your test command, and this can bypass the executables that RVM prefers. The solution:

    git bisect run sh -c 'source ~/.rvm/scripts/rvm && bundle exec rspec spec/models/whoops.rb'
