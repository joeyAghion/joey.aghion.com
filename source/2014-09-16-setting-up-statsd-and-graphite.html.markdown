---
title: Setting up StatsD and Graphite
date: 2014-09-16 19:45 -04:00
tags: programming, vagrant, berkshelf, aws
---

[Artsy](https://artsy.net) depends on [statsd](https://github.com/etsy/statsd) and [graphite](http://graphite.wikidot.com/) every day. When our graphite server failed recently, I took the opportunity to revisit an [old post](/tracking-application-metrics-with-statsd) about the simplest possible steps for setting up your own statsd and graphite server.

It turns out, the tools have improved quite a bit since then. [This post](https://coderwall.com/p/pb74zq) provided an outline in which you only need a Berksfile (to specify cookbooks) and a Vagrantfile (to drive the provisioning). Its settings and cookbooks were slightly out of date, so I made some modifications.

In _Berksfile_:

    source "https://supermarket.getchef.com"

    cookbook 'apt'
    cookbook 'statsd', git: 'https://github.com/librato/statsd-cookbook.git'
    cookbook 'graphite', git: 'https://github.com/hectcastro/chef-graphite.git'

In _Vagrantfile_:

    VAGRANTFILE_API_VERSION = "2"

    Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
      config.omnibus.chef_version = :latest
      config.vm.box = "dummy"
      config.vm.box_url = "https://github.com/mitchellh/vagrant-aws/raw/master/dummy.box"
      
      # Provider
      config.vm.provider :aws do |aws, override|
        aws.access_key_id = '...YOUR AWS_ACCESS_KEY_ID...'
        aws.secret_access_key = '...YOUR AWS_SECRET_ACCESS_KEY...'
        aws.keypair_name = 'your_keypair_name'
        aws.ami = "ami-d017b2b8" # Ubuntu 12.04.2 LTS us-east ebs-backed 64-bit (chef?)
        aws.region = "us-east-1"
        aws.instance_type = "m3.large"
        aws.security_groups = ["default", "statsd_graphite"]
        override.ssh.username = "ubuntu"
        override.ssh.private_key_path = "/path/to/your/keypair.pem"
        aws.tags = {
          'Name' => 'Statsd (Vagrant Provision)'
        }

        # Mount a 50 GB root partition rather than run out of space on the default
        aws.block_device_mapping = [
          {
            'DeviceName' => "/dev/sda1",
            'VirtualName' => "root",
            'Ebs.VolumeSize' => '50'
          }
        ]
      end

      # Provisioning
      config.berkshelf.enabled = true
      config.vm.provision :chef_solo do |chef|

        chef.add_recipe "apt"
        chef.add_recipe "statsd"
        chef.add_recipe "graphite"

        # We override the graphite cookbook's default storage_aggregation and storage_schemas settings according to Etsy's recommendations at https://github.com/etsy/statsd/blob/master/docs/graphite.md
        chef.json = {
          statsd: {
            legacyNamespace: false  # adopt new, preferred namespace scheme (all statsd metrics managed under "stats")
          },
          graphite: {
            storage_aggregation: [
              {
                min: {
                  pattern: "\\.lower$",
                  xFilesFactor: "0.1",
                  aggregationMethod: "min"
                }
              },
              {
                max: {
                  pattern: "\\.upper(_\\d+)?$",
                  xFilesFactor: "0.1",
                  aggregationMethod: "max"
                }
              },
              {
                sum: {
                  pattern: "\\.sum$",
                  xFilesFactor: "0",
                  aggregationMethod: "sum"
                }
              },
              {
                count: {
                  pattern: "\\.count$",
                  xFilesFactor: "0",
                  aggregationMethod: "sum"
                }
              },
              {
                count_legacy: {
                  pattern: "^stats_counts.*",
                  xFilesFactor: "0",
                  aggregationMethod: "sum"
                }
              },
              {
                default_average: {
                  pattern: ".*",
                  xFilesFactor: "0.3",
                  aggregationMethod: "average"
                }
              }
            ],
            storage_schemas: [
              {
                stats: {
                  priority: "100",
                  pattern: "^stats.*",
                  retentions: "10s:6h,1min:6d,10min:1800d"
                }
              },
              {
                catchall: {
                  priority: "0",
                  pattern: ".*",
                  retentions: "60s:5y"
                }
              }
            ]
          }
        }
      end
    end

What you'll need to do:

1. Download and install [Vagrant](http://www.vagrantup.com/downloads.html)
2. Download and install [ChefDK](https://downloads.getchef.com/chef-dk)
3. Create an AWS keypair
4. Create a `statsd_graphite` AWS security group that allows SSH and HTTP access, as well as UDP access to port 8125 (for statsd)
5. Update the `access_key_id`, `secret_access_key`, `keypair_name`, and `private_key_path` values in the Berksfile
6. Run the following commands:
```
vagrant plugin install vagrant-berkshelf vagrant-omnibus vagrant-aws
vagrant up --provider aws
```

That's it!


