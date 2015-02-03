---
layout: post
title: Docker Jenkins Slave
description: "Building docker containers to use as Jenkins slaves"
modified: 2015-01-14
category: continuous-delivery
tags: [ansible, jenkins, docker]
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->

## Jenkins Docker Slave

The aim of the docker plugin is to be able to use a docker host to dynamically provision a slave, run a single build, then tear-down that slave.<br/>
br/>
This post will cover the different aspects of this and how to go about debugging it.br/>
br/>
As we are using RHEL6 in production the slave will be based on Centos 6 as its the closest base image.


## Useful resources

* [Docker Plugin](https://wiki.jenkins-ci.org/display/JENKINS/Docker+Plugin)<br/>

## Building the container with Ansible

- TODO

## SSH Demon configuration

The Docker container needs to run a sshd. Jenkins then treats the running container like a normal box. There are a number of changes that need to happen to the default **openssh-server** installation **sshd_config**

* Generate ssh_host_dsa_key & ssh_host_rsa_key *(this may be handled when the service is started)*
* Enable public key authentication

{% highlight bash%}
RSAAuthentication yes
PubkeyAuthentication yes
AuthorizedKeysFile	.ssh/authorized_keys
{% endhighlight %}

* Disable PAM authentication
{% highlight bash%}
UsePAM no
#UsePAM yes
{% endhighlight %}

-

## Debugging the built container

To validate the keys and sshd configuration is working before we connect Jenkins we should try and connect to the container<br/><br/>

On the host VM start the container and open up the ssh port and start the sshd demon and syslog<br/>

{% highlight bash%}
docker run --rm -p 49000:22 -it centos-slave:latest bash

[root@fc80963729ad /] service sshd start
Starting sshd:                                             [  OK  ]
[root@fc80963729ad /] service rsyslog start
Starting system logger:                                    [  OK  ]

{% endhighlight %}

Next we find out the IP of the running container and ssh to it as the builder user *(our Jenkins user)*<br/>

{% highlight bash%}
[root@keyst020 devadmin] docker ps -a
CONTAINER ID        IMAGE                 COMMAND             CREATED             STATUS              PORTS                   NAMES
fc80963729ad        centos-slave:latest   "bash"              49 seconds ago      Up 48 seconds       0.0.0.0:49000->22/tcp   happy_sammet

[root@keyst020 devadmin] sudo docker inspect fc80963729ad | grep -i ipa
        "IPAddress": "172.17.0.19",
[root@keyst020 devadmin]
[root@keyst020 devadmin]


[builder@keyst020 ~]$ ssh -p 22 builder@172.17.0.19
Last login: Tue Feb  3 11:46:32 2015 from 172.17.42.1

[builder@fc80963729ad ~]$

{% endhighlight %}


