---
title: Heroku with a side of EC2
date: 2012-02-01
tags: chef, delayed_job, programming, ruby
---

I [posted](http://artsy.github.com/blog/2012/01/31/beyond-heroku-satellite-delayed-job-workers-on-ec2) on the [Artsy Engineering blog](http://artsy.github.com) about supplementing our Heroku web app with EC2-hosted Delayed Job workers:

> Artsy engineers are big users and abusers of Heroku. It's a neat abstraction of server resources, so we were conflicted when parts of our application started to bump into Heroku's limitations. While we weren't eager to start managing additional infrastructure, we found that--with a few good tools--we could migrate some components away from Heroku without fragmenting the codebase or over-complicating our development environments.
> 
> There are a number of reasons your app might need to go beyond Heroku. It might rely on a locally installed tool (not possible on Heroku's locked-down servers), or require heavy file-system usage (limited to tmp/ and log/, and not permanent or shared). In our case, the culprit was Heroku's 512 MB RAM limit--reasonable for most web processes, but quickly exceeded by the image-processing tasks of our delayed_job workers. We considered building a specialized image-processing service, but decided instead to supplement our web apps with a custom EC2 instance dedicated to processing background tasks. We call these servers "satellites."

We use [Chef](http://www.opscode.com/chef) and [Fog](http://fog.io) to simplify the servers' administration. Take a look at [the source](https://github.com/joeyAghion/satellite_setup), and see [the original post](http://artsy.github.com/blog/2012/01/31/beyond-heroku-satellite-delayed-job-workers-on-ec2) for all the gory details. Feedback welcome!
