---
title: Let console_color keep you out of trouble
date: 2013-11-22 09:38 -05:00
tags: programming, dev-ops, rails
---

Who hasn't mistakenly typed into a production Rails console thinking it was development?

Introducing [console_color](https://github.com/joeyAghion/console_color), so you're never unsure of your environment. To install, add `console_color` to your Gemfile:

    gem 'console_color'

Your console will look like:

![production console](http://f.cl.ly/items/3U2M3c1c230s0S2X3n22/Screen%20Shot%202013-11-22%20at%2012.06.43%20AM.png)

I.e., DANGER. Or:

![development console](http://f.cl.ly/items/3j1o3w3N1E382a211d1P/Screen%20Shot%202013-11-21%20at%2011.48.18%20PM.png)

Sure--it's frivolous. But maybe it'll save you some trouble.

[Visit console_color on GitHub for more details.](https://github.com/joeyAghion/console_color)
