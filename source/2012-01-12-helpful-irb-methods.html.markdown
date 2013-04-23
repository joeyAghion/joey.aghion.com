---
title: Helpful IRB methods
date: 2012-01-12 7:38
tags: programming, ruby
---

My `~/.irbrc` file contains these helpers for performing sql queries and outputting raw database (SQL or MongoDB) queries to the console. Just add them to your home directory's .irbrc file to have these methods available in IRB or the Rails console.

<!-- from https://gist.github.com/joeyAghion/1371971 -->

    # Output ActiveRecord-generated queries to the console
    def log_to_console
      ActiveRecord::Base.logger = Logger.new(STDOUT)
      reload!
    end

    # Execute arbitrary SQL selects
    def sql(query)
      ActiveRecord::Base.connection.select_rows(query)
    end

    # Output Mongoid-generated queries to the console
    def log_mongo_to_console
      # For mongoid 2:
      #   Mongoid.config.logger = Logger.new(STDOUT)
      Mongoid.logger = Logger.new(STDOUT)
      Moped.logger = Mongoid.logger
    end

    # The list of methods to which obj responds.
    def m(obj)
      obj.methods.sort - Object.methods
    end
