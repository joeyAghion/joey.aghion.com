---
title: Simple 301 redirects
date: 2013-07-12 10:08 -04:00
tags: programming, github
---

When I switched from [halfamind.aghion.com](http://halfamind.aghion.com) to the more personal [joey.aghion.com](http://joey.aghion.com) for this site, I needed a way to permanently redirect all requests for the old hostname to the new one. [Github pages](http://pages.github.com) (where this is hosted) only supports a single hostname and doesn't allow for redirects. You could redirect [in javascript](http://stackoverflow.com/questions/9276817/301-redirect-for-site-hosted-at-github) or with a meta-tag, but I wanted to guarantee that search engines would respect the new location.

[Rerouter](http://github.com/joeyAghion/rerouter) is a simple rack app that accepts a map of source and destination hostnames. When it receives a request matching a source hostname, it issues a 301 (permanent) redirect to the new hostname while preserving the requested path and querystring.

The complete source is incredibly simple.

### config.ru


    require 'rack/rewrite'

    # Expects ENV['REDIRECTS'] to be a ruby hash of source => destination hostnames. E.g.:
    #   "{'old.domain.com' => 'new.domain.com'}"
    REDIRECTS = eval(ENV['REDIRECTS'] || '') || {}

    use Rack::Rewrite do

      REDIRECTS.each do |from, to|
        r301 %r{.*}, "http://#{to}$&", if: -> (env) { env['SERVER_NAME'] == from }
      end
  
    end

    # Fall back to default app (empty).
    run -> (env) { [200, {}, []] }


### Steps for setting up your own rerouter on Heroku:


    git clone https://github.com/joeyAghion/rerouter.git
    cd rerouter
    gem install heroku
    heroku apps:create
    git push heroku master
    heroku config:set REDIRECTS="{'old.domain.com'=>'new.domain.com'}"
    heroku domains:add old.domain.com

Then, update the DNS of `old.domain.com` to be a CNAME pointing to the new heroku app.
