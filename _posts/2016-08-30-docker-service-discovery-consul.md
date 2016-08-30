---
layout: post
title: Docker Swarm Service Discovery
description: "Swarm service discovery and Consul."
modified: 2016-08-30
category: devops
tags: [devops,docker,swarm, ansible, consul]
---

<section>
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->


## Service discovery in 1.12

Prior to version 1.12 of docker it was necessary to use external tools and services to provide service discovery for the containers running either directly with the Daemon or in the swarm.

Applications like **Consul**, **Zookeeper**, **etcd** and **serf** were used in combination with registration containers like **registrator**.

Version 1.12 of docker introduces multi-host / multi-container networking and [service discovery](https://blog.docker.com/2016/06/docker-1-12-built-in-orchestration/). Containers that are associated with an overlay network can communicate with easy other using the name of the container as the hostname. Requests are balanced using round-robin between the replicas.

### Reasons for still using external service discovery.

- v1.12 is very new and external discovery tools may well be already invested in
- External discovery - Docker engine service discovery is internal to the containers only.
- Consul provides k/v stores, DNS and health checks as well as service discovery
- Docker services don't work with registrator (at the moment) due to the way ports are managed now.


## Consul

### Why choose Consul?

As mentioned above Consul provides much more than just service discovery. It also rates highly against tools such as Zookeeper & etcd. For more information I would recommend reading [The DevOps 2.0 Toolkit](https://leanpub.com/the-devops-2-toolkit) where these tools are evaluated.

### Registrator

Registrator is a docker container that listens to the docker daemon for events and registers/de-registers the containers as services in Consul.

> There is currently an [open](https://github.com/gliderlabs/registrator/issues/443) defect preventing docker services from being detected due to how the ports are mapped.


### [Consul-notifier](https://github.com/jamesdmorgan/vagrant-ansible-docker-swarm/tree/master/consul-notifier)

Since registrator does not currently handle services I decided to create a very simple python script that emulates registrator listening to the docker daemon events but uses port environment variables to register / de-register services.

I am hoping that registrator will be fixed shortly and I can revert to using just that.



