---
title: Profile Redis memory usage by key pattern
date: 2010-10-13 7:43
tags: programming, redis, ruby
---

At [Weplay](http://www.weplay.com), we use [Redis](http://code.google.com/p/redis/) quite a bit. As a Redis database grows, though, it can be difficult to understand how the space is being used. This script tries to identify which types of keys are occupying the most space in your Redis databases. To do so, it takes a random sample of keys and collapses them into similar patterns (basically by ignoring the numeric values).

Sample output:


    Profiling "db0"...
    --------------------
    ---
    - - User:#:cached_preferences
      - keys: 1715
        size: 36015
        percent: 54.11%
    - - Poll:#:results
      - keys: 555
        size: 27750
        percent: 41.69%
    - - Group:#:member_ids
      - keys: 1645
        size: 1645
        percent: 2.47%
    - - User:#:cached_follower_ids
      - keys: 577
        size: 1154
        percent: 1.73%
    
    Overall statistics:
    --------------------
    used_memory:1102224
    used_memory_human:1.05M

The script: [http://gist.github.com/624334](http://gist.github.com/624334)

    #!/usr/bin/env ruby

    # Evaluates a sample of keys/values from each redis database, computing statistics for each key pattern:
    #   keys: number of keys matching the given pattern
    #   size: approximation of the associated memory occupied (based on size/length of value)
    #   percent: the proportion of this 'size' relative to the sample's total
    #
    # Copyright Weplay, Inc. 2010. Available for use under the MIT license.


    require 'rubygems'
    require 'redis'
    require 'yaml'

    SAMPLE_SIZE = 10_000  # number of keys to sample from each db before computing stats


    # Naive approximation of memory footprint: size/length of value.
    def redis_size(db, k)
      t = db.type(k)
      case t
        when 'string' then db.get(k).length
        when 'list'   then db.lrange(k, 0, -1).size
        when 'zset'   then db.zrange(k, 0, -1).size
        when 'set'    then db.smembers(k).size
        else raise("Redis type '#{t}' not yet supported.")  # TODO accommodate more types
      end
    end

    def array_sum(array)
      array.inject(0){ |sum, e| sum + e }
    end

    def redis_db_profile(db_name, sample_size = SAMPLE_SIZE)
      db = Redis.new(:db => db_name)
      keys = []
      sample_size.times { |i| keys << db.randomkey }
      key_patterns = keys.group_by{ |key| key.gsub(/\d+/, '#') }
      data = key_patterns.map{ |pattern, keys|
        [pattern, {'keys' => keys.size, 'size' => array_sum(keys.map{ |k| redis_size(db, k) })}]
      }.sort_by{ |a| a.last['size'] }.reverse
      size_sum = data.inject(0){|sum, d| sum += d.last['size'] }
      data.each { |d| d.last['percent'] = '%.2f%' % (d.last['size'].to_f*100/size_sum) }
    end

    db_names = `redis-cli info | grep ^db[0-9]`.split.map{ |line| line.scan(/^db\d+/).first }
    db_names.each do |name|
      puts "\nProfiling \"#{name}\"...\n#{'-'*20}"
      y redis_db_profile(name)
    end

    puts "\nOverall statistics:\n#{'-'*20}"
    puts `redis-cli info | grep memory`
