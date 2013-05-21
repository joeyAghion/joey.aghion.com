---
title: Eliminating TextMate Lag
date: 2013-05-21 12:12 -04:00
tags: programming, tools
---

I'm still using [TextMate 1](http://macromates.com/), since 2.0 has been buggy for me. If you're in the same boat, you might share my frustration with the recurring lag that TextMate experiences when switching focus between applications. TextMate tries to reload the entire project's file state on focus, and this can be noticeably slow.

These tips have eliminated the problem for me. Feel free to mix and match:

### Avoid loading the entire project directory

If you're working with a Rails project, you can avoid loading the heavy `log/` and `tmp/` directories with the `mater` alias. Just put this in one of your shell start-up scripts:

    alias mater="ls | grep -Ev 'log|tmp' | xargs mate"

Adjust for other project types.

### Disable project tree refresh with ReMate

[The ReMate plugin](http://ciaranwal.sh/remate/) adds a _Disable Refresh on Regaining Focus_ item to the _Window_ menu. Download and extract the plugin, double click to add to TextMate, and relaunch TextMate. Then, check the new menu option to disable the project refresh behavior entirely. (You can always temporarily uncheck it to refresh the project state.)
