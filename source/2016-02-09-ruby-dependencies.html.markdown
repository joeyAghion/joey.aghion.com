---
title: Ruby dependencies
date: 2016-02-09 10:34 -05:00
tags: programming, ruby
---

Mike Perham published an excellent rant pleading with developers (and especially developers of Ruby gems) to [Kill Your Dependencies](http://www.mikeperham.com/2016/02/09/kill-your-dependencies/).

This is a common enough idea (see [A quick note on dependencies in Ruby on Rails projects](http://www.fngtps.com/2013/a-quick-note-on-minimal-dependencies-in-ruby-on-rails/)), and anyone responsible for maintaining a large, complex, or elderly codebase has probably arrived at the conclusion independently. Still, a reminder is in order. I've certainly been guilty myself of leaning a little _too_ heavily on the casual contributions of others.

To his excellent list, I'll add a few practices that we've started applying at [Artsy](https://www.artsy.net):

**Each addition to your Gemfile should include a comment explaining the choice.** Instead of the Gemfile being a simple list, think of it as a table with 3 columns: gem name, [optional] version constraint, and _justification_. A sample from one of our recent projects:

    gem 'unicorn'  # avoid timeouts on single web process
    gem 'stringex'  # clean up extended characters in email names
    gem 'newrelic_rpm'  # track errors, transactions
    gem 'gridhook'  # handle sendgrid event notifications

**Avoid adding gems that monkeypatch external code or behave differently based on what other dependencies have been loaded.** And especially, avoid _writing_ such gems. When some enterprising developer tries to trace the dependencies' usage years later in the hope of removing or replacing them, full-text searches won't reveal a thing. Stay within your namespace if at all possible.
