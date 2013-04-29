---
title: Building a blog with Middleman and GitHub pages
date: 2013-04-28
tags: programming, middleman, github
---

[Posterous is shutting down](http://blog.posterous.com/thanks-from-posterous), so I was finally forced to figure out a better blogging solution. Since my [day job](http://artsy.net) revolves around building and maintaining dynamic sites, that was exactly what I _didn't_ want for my personal site. Enter [Middleman](http://middlemanapp.com/), the super-simple **static** site engine. Best of all, it uses a toolchain that's very familiar to Ruby developers.


## Getting Started

The [middleman](https://github.com/middleman/middleman) gem supports static sites of all sorts, but since this site is primarily a blog, I reached straight for the [middleman-blog](https://github.com/middleman/middleman-blog) extension.

    gem install middleman-blog
    middleman init myblog --template=blog

This generates the following folder structure:

    myblog
     |- .gitignore
     |- config.rb
     |- Gemfile
     |- Gemfile.lock
     |- source
        |- 2012-01-01-example-article.html.markdown
        |- calendar.html.erb
        |- feed.xml.builder
        |- images
        |- index.html.erb
        |- javascripts
        |- layout.erb
        |- stylesheets
        |- tag.html.erb

Those files and folders are almost self-explanatory, but to call out a few:

* **config.rb** holds all of middleman's configuration, including a block for blog-specific settings.
* **source** holds just that---the source files. These are later compiled (according to the middleman settings you've chosen) into a **build** directory that contains the final site's files.
* **layout.erb** is the main site layout. ERB is the default, but middleman supports [many other templating engines](http://middlemanapp.com/templates/).

## Creating an article

Actually, there's an example article included. But, to create our own:

    bundle exec middleman article "Launching my blog"

This creates a new source file at `source/2013-04-28-launching-my-blog.html.markdown` with the appropriate YAML-formatted "frontmatter." You can add comma-delimited tags and compose the post in markdown:

    ---
    title: Launching my blog
    date: 2013-04-28 03:26 UTC
    tags: posterous, github pages
    ---

    Buh bye posterous!

After starting up the development server with `bundle exec middleman`, we can open http://localhost:4567 to check our work:

<img src="/images/2013-04-28-middleman-screenshot.png" alt="Initial middleman screenshot" />

It's unsightly for now, but---when deployed as static HTML---it's very, very fast.

## Deploying

To compile the full site's static content:

    bundle exec middleman build

Note that the `build` folder containing the compiled site is _not_ checked in (it's listed in the default `.gitignore` file).

At this stage, that folder's contents could be hosted just about anywhere. We'll deploy our site to [github pages](http://pages.github.com/) with help from the [middleman-gh-pages](https://github.com/neo/middleman-gh-pages) gem. First, we add it to the Gemfile:

    gem "middleman-gh-pages"

And require it in a `Rakefile`:

    require 'middleman-gh-pages'

After bundle-installing again, we'll have a new `publish` rake task that pushes the static site to a special `gh-pages` branch on the same remote as the source. (If you haven't made a github repo for this project yet, now would be a good time.)

    bundle exec rake publish

This last step is worth explaining. The "gh-pages" branch is specially named so that Github will publish its contents to your site at _username.github.io/projectname_. It's an "orphan" branch (meaning it has no parents). I.e., it's completely divorced from your master branch's content. The master branch contains your source files, and the static files compiled to the `build` subdirectory become the root folder's content in the `gh-pages` branch. Luckily, you don't have to manage the `gh-pages` branch directly; the `publish` task takes care of compiling your site and creating the corresponding commits on `gh-pages`.

Finally, setting up a nice domain for your site is as simple as updating DNS records and adding a `CNAME` file to the root of your site (or, in middleman's case, the `source` directory) with the specified domain. [See github's more detailed instructions.](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)

Styling the site is left as an exercise, or feel free to borrow from [this site's source](http://github.com/joeyAghion/joey.aghion.com).

## More information

* Advanced features, community extensions, and more at [the middleman site](http://middlemanapp.com)
* [middleman-blog extension](http://middlemanapp.com/blogging/)
* ["Pretty" URLs](http://middlemanapp.com/pretty-urls/)
