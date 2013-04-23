---
title: Bash aliases and helpers
date: 2012-01-12 7:43
tags: programming, tools
---

My ~/.bash_profile file contains these helpers and aliases.

<!-- from https://gist.github.com/joeyAghion/1371971 -->

    # Display current git branch, nicely colored, in the prompt (with a * if there are changes)
    function parse_git_dirty {
      [[ $(git status 2> /dev/null | tail -n1) != "nothing to commit (working directory clean)" ]] && echo "*"
    }
    function parse_git_branch {
      git branch --no-color 2> /dev/null | sed -e '/^[^*]/d' -e "s/* \(.*\)/[\1$(parse_git_dirty)]/"
    }
    export PS1='\u:\[\033[31;40m\]\w\[\033[0;33m\]$(parse_git_branch)\[\e[0m\]$ '

    export EDITOR='mate -w'

    export PATH=$PATH:/usr/local/rvm/bin:~/bin
     [[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"

    # colorize terminal
    export CLICOLOR=1
    export LSCOLORS=GxFxCxDxBxegedabagaced

    # Find running processes, e.g.:
    #   $ psx chrome
    alias psx='ps aux | grep $1'


    # Recursive full text search, e.g.:
    #   $ g somestring app/
    alias g='grep -rn'
    export GREP_OPTIONS='--color=auto'

    # Output my current IP address (thanks @hmason), e.g.:
    #   $ whatsmyip
    alias whatsmyip='curl http://automation.whatismyip.com/n09230945.asp'


    # Open man pages in Preview (thanks @brynary), e.g.:
    #   $ pman grep
    pman () {
        man -t "${1}" | open -f -a /Applications/Preview.app
    }

    # Who hosts that site (thanks @jcn), e.g.:
    #   $ whohosts foursquare.com
    whohosts () {
      whois `/opt/local/lib/mysql5/bin/resolveip -s $*`;
    }

    # exclude tmp/ and log/ when opening a rails project in textmate
    alias mater="ls | grep -Ev 'log|tmp' | xargs mate"

    # via @desandro http://desandro.github.com/dropshado.ws/
    alias glog="git log --format='%Cgreen%h%Creset %C(cyan)%an%Creset - %s' --graph"

