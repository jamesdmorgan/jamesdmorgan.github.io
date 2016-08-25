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

As discussed in the previous [post](http://jamesdmorgan.github.io/2016/opsdev/) I want to create a local enviroment that utilises many of the tools and architecture needed in production, such as **High Availability**, **Fault Tolerance**, **Service Discovery**, **Data Persistence**, **Centralised Logging** and  **Monitoring**

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

The [Vagrantfile](https://github.com/jamesdmorgan/vagrant-ansible-docker-swarm/blob/master/Vagrantfile) brings up 6 VirtualBox virtual machines.

3 Managers and 3 Worker boxes. There are 3 managers so we can test swarm & consul clustering. It needs to be an odd number so Quorum can be achieved. Its possible to run 1 manager and 1 worker if the resources are not there. Though the Ansible groups will need to be re-balanced to move monitoring and logging

```ruby
MANAGERS = 3
WORKERS = 3
ANSIBLE_GROUPS = {
  "managers" => ["manager[1:#{MANAGERS}]"],
  "workers" => ["worker[1:#{WORKERS}]"],
  "elk" => ["manager[2:2]"],
  "influxdb" => ["manager[3:3]"],
  "all_groups:children" => [
    "managers",
    "workers",
    "elk",
    "influxdb"]
}
```

The virtual machines are running Centos7. Many of the production environments I have previously worked on used either Redhat or Centos. For machines that are only running Docker I may look to use a more lightweight version of Linux. Many of the small containers use Alpine Linux due to is small footprint. So that could be a good option for the actual [virtual machine](https://github.com/maier/vagrant-alpine).

### Vagrant plugins

- vagrant-host-shell
- vagrant-alpine
- vagrant-vbguest

```bash
> vagrant plugin install vagrant-alpine
```

### Running Vagrant

To provision the virtual machines and install and configure the Swarm and overlay network

```bash
> vagrant up --provision
```

Each area is managed by its own Ansible playbook which can be invoked using

```bash
> vagrant provision --provision-with monitoring
```

If you want the entire thing run consul, monitoring, logging, apps

## Ansible


## Docker Swarm