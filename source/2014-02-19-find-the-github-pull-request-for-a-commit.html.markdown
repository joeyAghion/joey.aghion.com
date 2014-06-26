---
title: Find the github pull request for a commit
date: 2014-02-19 11:21 -05:00
tags: programming, github, bash
---

Inspired by the post [Every line of code is always documented](http://mislav.uniqpath.com/2014/02/hidden-documentation/), I upgraded my development environment with a few new tools:

* [GitCommitMsg](https://github.com/cbumgard/GitCommitMsg) plug-in for Sublime Text - provides easy access to the entire commit history for selected lines.
* The `pr_for_sha` command below (stored in my ~/.bash_profile) - If you employ [github flow](http://scottchacon.com/2011/08/31/github-flow.html), this command pops open the github page for the pull request where a given commit originated:

<pre>
function pr_for_sha {
  git log --merges --ancestry-path --oneline $1..master | grep 'pull request' | tail -n1 | awk '{print $5}' | cut -c2- | xargs -I % open https://github.com/$2/${PWD##*/}/pull/%
}
</pre>

Now you can call pr_for_sha commit_sha upstream_name and it will open the pull request in your browser.

And if you're not already using them, you should of course install these Sublime Text plug-ins (via [Package Control](https://sublime.wbond.net/)):
* [Git](https://github.com/kemayo/sublime-text-git) - integrates the ever-useful **Git Blame** command.
* [GitGutter](http://www.jisaacks.com/gitgutter) - indicates added, removed, and modified lines right in your editor.
