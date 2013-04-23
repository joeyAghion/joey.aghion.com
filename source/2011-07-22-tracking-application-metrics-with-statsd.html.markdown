---
title: Tracking Application Metrics with Statsd
date: 2011-07-22
tags: chef, programming, ruby, startups
---

At [Weplay](http://www.weplay.com), I often find myself wanting a dead-simple way to track an application-level metric or event. That's why I found Etsy's post about Statsd ([Measure Anything, Measure Everything](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/)) very appealing. Statsd is a node.js daemon that receives notifications of application events and sends the data to [Graphite](http://graphite.wikidot.com/) for graphing.

[Skip to the chef recipes](https://github.com/joeyAghion/statsd_setup)

Three characteristics in particular got my attention:

- Minimal connection overhead for the calling app, since data is sent to the statsd daemon via UDP (fire and forget)
- Bounded data storage needs, since Graphite stores the ever-growing metrics data in decreasing resolution over time ([like RRDtool but different](http://graphite.wikidot.com/faq#toc8))
- Practically no management of metrics is necessary--the first time a metric is triggered, statsd will start tracking it and Graphite will start making the graph available, with results organized into "folders" according to the dots in their names (e.g., `system.deploys`, `cart.products_added`, etc.)

![Log-in successes and failures, courtesy of Etsy's Code as Craft blog](http://etsycodeascraft.files.wordpress.com/2011/02/logins2.png?w=500&amp;h=300 "log-in successes and failures")

_(Graph from [Measure Anything, Measure Everything](http://codeascraft.etsy.com/2011/02/15/measure-anything-measure-everything/).)_

Graphite offers a competent web UI and loads of options for data manipulation and presentation, including overlaying different streams of data and returning raw metrics. And because Graphite's graphs are completely specified by URLs, they can easily be embedded in your other dashboards and reporting tools.

In a related vein, [Matt Insler](http://www.mattinsler.com/) of [Signpost](http://www.signpost.com) recently posted about [building a tracking infrastructure on MongoDB](http://www.mattinsler.com/signpost-tracking-analytics-mysql-mongodb/). Both approaches use a "fire and forget" method of emitting the tracked data, but while Statsd's metrics are stripped down streams of counts and times, Matt opted to store exhaustive request and event data. One offers automatic graphing, the other powerful ad hoc querying--interesting and perhaps complementary strategies.

## Setting up statsd ##

Here are my steps for spawning and configuring a simple EC2 instance with statsd and graphite:

If you haven't already...

- download and install the [EC2 API tools](http://aws.amazon.com/developertools/351) (e.g., into `~/tools`)
- generate a X.509 certificate (both public `cert-...pem` and `private pk-...pem` files) on the [AWS Security Credentials page](https://aws-portal.amazon.com/gp/aws/developer/account/index.html?action=access-key)
- generate a keypair (`statsd_setup.pem`) in the [AWS Management Console](https://console.aws.amazon.com/)

I store the key files in `~/.ec2/` locally.

    $ sudo chmod 600 ~/.ec2/statsd_setup.pem  # set appropriate permissions on private key file

The EC2 API tools require certain environment variables to be set. I put the following in a local file (e.g., `~/bash/statsd_setup.sh`):

    export EC2_HOME=~/tools/ec2-api-tools-1.4.3.0
    export EC2_PRIVATE_KEY=~/.ec2/pk-statsd_setup.pem
    export EC2_CERT=~/.ec2/cert-statsd_setup.pem
    export JAVA_HOME=/System/Library/Frameworks/JavaVM.framework/Versions/CurrentJDK/Home
    export PATH=$PATH:$EC2_HOME/bin

...and load them:

    $ source ~/bash/statsd_setup.sh

Finally, start the instances and modify some basic security settings:

    $ ec2-run-instances ami-06ad526f -k statsd_setup  # start a new instance with a recent Ubuntu 11 image
    $ ec2-authorize default -p 22                     # permit SSH
    $ ec2-authorize default -p 80                     # permit HTTP
    $ ec2-authorize default -p 8125 -P udp            # statsd will listen here

I use [spatula](http://github.com/trotter/spatula) to apply [chef](http://www.opscode.com/chef/) recipes to simple environments. Chef recipes and another set of these instructions can be cloned from [my `statsd_setup` github repo](http://github.com/joeyAghion/statsd_setup) as follows:

    $ git clone git@github.com:joeyAghion/statsd_setup.git
    $ cd statsd_setup

Run `ec2-describe-instances` to view the new instance's external IP, then substitute it in the following commands:

    $ spatula prepare ubuntu@184.72.76.150 --identity ~/.ec2/statsd_setup.pem    # set up chef prerequisites
    $ spatula cook ubuntu@184.72.76.150 node --identity ~/.ec2/statsd_setup.pem  # apply recipes for statsd and graphite

At this point, statsd and Graphite are ready to start tracking metrics. Visit your instance in a browser to access Graphite's web interface.

## Tracking metrics from your Ruby or Rails app ##

I added the `statsd-ruby` gem to my app's `Gemfile` and created an initializer with something like:

    require 'statsd'
    $statsd = Statsd.new('10.71.75.149')  # substitute your host

Then, tracking stats is as simple as:

    $statsd.increment('deploys')                  # increment a stat
    $statsd.decrement('usage.active_users')       # decrement a stat
    $statsd.count('cart.products_added', 3)       # track an arbitrary stat
    $statsd.count('cart.products_added', 3, 0.1)  # track a stat, sampling 10%
    $statsd.timing('partners.twitter_api', 650)   # track a time value (in ms)
    $statsd.time('partners.facebook_api') {...}   # track time spent executing the given block

You can always terminate the instance with `ec2-terminate-instance i-17fc3c76` (substituting the appropriate instance id).
