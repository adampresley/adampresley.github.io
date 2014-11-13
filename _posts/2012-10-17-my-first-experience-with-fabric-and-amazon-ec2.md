---
layout: post
title: My First Experience with Fabric and Amazon EC2
date: 2012-10-17 13:03:00
author: Adam Presley
status: Published
tags: python deployment
slug: my-first-experience-with-fabric-and-amazon-ec2
---
I've been working on a few projects and I have been in the process of
moving them to Amazon EC2. Up until this point I have been writing Bash
shell scripts, connecting to my servers and EC2 instances via SSH, then
running the shell scripts manually. Then I learned about [Fabric](http://docs.fabfile.org/), and
I wish I had seen this sooner. Fabric is a command-line tool and Python
library for executing local and remote shell commands. Fabric can also
connect to remote servers over SSH and execute commands. This basically
renders me antique. :) I'm OK with this. In my setup I have a deployment
key on each EC2 instance that is used to checkout the Git repository.
This repository is the home of the web application. From here I would
usually update a file named **config.py** to turn off *Debug* which has
my application bind to a different port, etc... I also run a script
named **bundle.py** that takes all my CSS and JS files, combines and
minifies them, then changes the includes in the main layout template to
point to these files instead of the individual ones. Now, however, I can
automate those tasks with a single command using Fabric.   
  
Below is an example of how I am using Fabric and Boto (for Amazon) to
iterate over my EC2 instances, reset the repository (undoing all local
changes), pull down the latest code, fix the config file and bundle.

{% highlight python %}
from __future__ import with_statement

import fabric, boto
import boto.ec2, time
from fabric.api import *
from fabric.colors import green, yellow
from boto.ec2.connection import EC2Connection

ACCESS_KEY_ID = "PUT YOUR KEY HERE"
SECRET_ACCESS_KEY = "PUT YOUR ACCESS KEY HERE"

REPOS = [
    ("/path/to/repo", "origin", "master")
]


def _connect():
    return EC2Connection(ACCESS_KEY_ID, SECRET_ACCESS_KEY)

def _getInstances():
    connection = _connect()
    reservations = connection.get_all_instances()
    instances = []

    print ""
    print green("** Getting instances **")

    for reservation in reservations:
        for instance in reservation.instances:
            print "Instance: %s (%s) @ %s" % (instance.id, instance.state, instance.public_dns_name)
            instances.append(instance.public_dns_name)

    return instances


def _gitReset():
    print ""
    print yellow("Resetting repository...")
    run("cd %s && git checkout -f" % env.repo)


def _gitPull():
    print ""
    print yellow("Updating code...")
    run("cd %s && git pull %s %s" % (env.repo, env.parent, env.branch))


def _fixConfigFile():
    print ""
    print yellow("Updating config file...")
    run("sed -i 's/DEBUG\s=\sTrue/DEBUG = False/g' /path/to/repo/config.py")


def _bundle():
    print ""
    print yellow("Bundling...")
    run("cd /path/to/repo/bin && python ./bundle.py")


def production():
    env.hosts = []
    env.user = "ubuntu"
    env.key_filename = [ "super-secret-key-pair-file.pem" ]

    #
    # Get the DNS addresses for all the instances on this account
    #
    env.hosts = _getInstances()


def updateServers():
    require("hosts", provided_by = [ production ])

    print ""
    print green("** Updating Servers **")

    for repo, parent, branch in REPOS:
        env.repo = repo
        env.parent = parent
        env.branch = branch

        print yellow("%s at %s/%s" % (repo, parent, branch))

        _gitReset()
        _gitPull()
        _fixConfigFile()
        _bundle()
{% endhighlight %}