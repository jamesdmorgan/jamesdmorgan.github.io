---
layout: post
title: Vagrant Docker Swarm
description: "Automated Docker Swarm cluster"
modified: 2016-08-14
category: devops
tags: [devops,vagrant,docker,ansible]
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


## Overview

As discussed in the previous post I want to create a local enviroment that utilises many of the tools and architecture needed in production, such as **High Availability**, **Fault Tolerance**, **Service Discovery**, **Data Persistence**, **Centralised Logging** and  **Monitoring**

The underlying architecture for experimenting with the different tools will be the latest version of Docker Swarm (1.12) running on a number of VirtualBox virtual machines.

> These blog posts will start with Vagrant and the swarm and iteratively add new components all via Ansible.

Every component in the system will be fully automated. I could have chosen to use **Docker Machine** to spin up the virtual machines but I chose to use Vagrant due to its clean integration with Ansible. Ansible then provisions each component in the system. In order to keep things modular the tools and tasks are grouped into logical areas

- Docker Swarm (Docker 1.12)
- Logging (centralised logging to ELK stack)
- Service Discovery (1.12 includes discovery but not external to the swarm)
- Monitoring
- Applications (simple applications to demo logging, monitoring and the swarm)

Each of these groups can be provisioned separately with Vagrant or directly with Ansible.

> The project can be found on [**github**](https://github.com/jamesdmorgan/vagrant-ansible-docker-swarm)

## Vagrant


## Ansible


## Docker Swarm