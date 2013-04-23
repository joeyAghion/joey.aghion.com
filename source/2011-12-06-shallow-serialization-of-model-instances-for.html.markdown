---
title: Shallow Serialization of Model Instances for Delayed_Job and MongoDB
date: 2011-12-06
tags: delayed_job, mongo, programming, ruby
---

Using [delayed_job](https://github.com/collectiveidea/delayed_job) and [MongoDB](http://www.mongodb.org/) (via [Mongoid](http://mongoid.org/))? Delayed_Job's _delay proxy_ is a convenient way to queue up background tasks on a model instance:

    user = User.find(id)
    user.delay.perform_time_consuming_task(args)

However, the model object is serialized and stored as YAML in the delayed_job record when called this way. It's verbose and unnecessary.

Introducing [delayed_job_shallow_mongoid](https://github.com/joeyAghion/delayed_job_shallow_mongoid), which short-circuits that serialization. When a delayed job is called on a model instance or a model instance is passed to one as an argument, a stub object is recorded instead of the fully serialized object. When the job is run, the stub is recognized and a _find_ is done to look up the underlying document(s). If a referenced model isn't found at that point, the job simply returns.

(This approach is based on similar optimizations for ActiveRecord in [popular delayed_job forks](https://github.com/tobi/delayed_job).)
