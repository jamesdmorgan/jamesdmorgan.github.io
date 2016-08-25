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

To log onto the boxes simply run

```bash
> vagrant ssh manager1
Last login: Thu Aug 25 11:27:47 2016 from 10.0.2.2
[vagrant@manager1 ~]$ sudo su
[root@manager1 vagrant]#
```

## Ansible

### Idempotency
One of Ansibles key features is that it's modules are (mostly) idempotent. I.e they can be re-run, if the desired effect is already achieved then it skips the task. Use of the shell modules causes a problem as its not idempotent if it changes the system.

Ansible has released many new docker modules with the v2 release though it doesn't support Docker 1.12 service, networking yet. I have therefore used **shell** to query the current state and only run **shell** if the network or service hasn't been created. This doesn't allow for restarting but we will have to live with that at the moment.

```yaml
    - name: Get existing services
      shell: >
        docker service ls --filter name={{ item.name }} | tail -n +2
      with_items: "{{ docker_services }}"
      register: services_result

    - set_fact:
        docker_current_services: "{{ services_result.results | map(attribute='stdout') | list | join(' ') }}"
```

The example above formats up the response so we can easily search it later.

The current docker command doesn't have any json options so we have to filter and strip the header

```bash
[root@manager1 vagrant]# docker service ls
ID            NAME      REPLICAS  IMAGE                COMMAND
3k1j6qxshlaq  collectd  global    collectd
63cjh5tbg8gr  influxdb  1/1       tutum/influxdb:0.12
```

As we are checking multiple apps with with_items the json returned will have a list of dictionaries. The new [map](http://jinja.pocoo.org/docs/dev/templates/#map) filter allows us to select a value from a list of dicts.

I then flatten this and turn it into a string which can be searched. We don't run the command if the service exists.

```yaml
  shell: >
    docker service create \
      --name {{ service_dict.name }} \
      --env "CONSUL_SERVICE_PORT={{ service_dict.service_port | default(80) }}" \
      --log-driver syslog \
      --log-opt tag={{ docker_syslog_tag }} \
      {{ service_dict.definition }}
  when: service_dict.name not in docker_current_services
```



## Docker Swarm