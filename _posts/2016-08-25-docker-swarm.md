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

As discussed in the previous [post](http://jamesdmorgan.github.io/2016/opsdev/) I want to create a local environment that utilises many of the tools and architecture needed in production, such as **High Availability**, **Fault Tolerance**, **Service Discovery**, **Data Persistence**, **Centralised Logging** and  **Monitoring**

The underlying architecture for experimenting with the different tools will be the latest version of Docker Swarm (1.12) running on a number of VirtualBox virtual machines.

> These blog posts will start with Vagrant and the swarm and iteratively add new components all via Ansible.

The status of the project is detailed on the [Changelog](https://github.com/jamesdmorgan/vagrant-ansible-docker-swarm/blob/master/CHANGELOG.md)

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

Ansible was chosen as it provides a simple, easily readable way to automate the installation and configuration of each component. Vagrant auto-generates the inventory simplifying things further. We can then reference the boxes by the group hostvars information. Each area has its own playbook and the components are all abstracted away in roles.

As mentioned above. Ansible can either by run via `vagrant provision` or directly. I have created the following alias

```bash
alias ansible-vagrant='PYTHONUNBUFFERED=1 ANSIBLE_FORCE_COLOR=true ANSIBLE_HOST_KEY_CHECKING=false ANSIBLE_SSH_ARGS='\''-o UserKnownHostsFile=/dev/null -o IdentitiesOnly=yes -o ControlMaster=auto -o ControlPersist=60s'\'' ansible-playbook --connection=ssh --timeout=30 --inventory-file=.vagrant/provisioners/ansible/inventory'
```

which allows me to run individual playbooks from the root project directory. Its slightly faster and you can add extra verbosity / limits etc.

```bash
> ansible-vagrant ansible/monitoring.yml -vvv
```

### Idempotency
One of Ansible's key features is that it's modules are (mostly) idempotent. I.e they can be re-run, if the desired effect is already achieved then it skips the task. Use of the shell modules causes a problem as its not idempotent if it changes the system.

Ansible has released many new docker modules with the v2 release though it doesn't support Docker 1.12 service, networking yet. I have therefore used **shell** to query the current state and only run **shell** if the network or service hasn't been created. This doesn't allow for restarting but we will have to live with that at the moment.

```yaml
{% raw %}
- name: Get existing services
  shell: >
    docker service ls --filter name={{ item.name }} | tail -n +2
  with_items: "{{ docker_services }}"
  register: services_result

- set_fact:
    docker_current_services: "{{ services_result.results | map(attribute='stdout') | list | join(' ') }}"
{% endraw %}
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
{% raw %}
shell: >
    docker service create \
      --name {{ service_dict.name }} \
      --env "CONSUL_SERVICE_PORT={{ service_dict.service_port | default(80) }}" \
      --log-driver syslog \
      --log-opt tag={{ docker_syslog_tag }} \
      {{ service_dict.definition }}
when: service_dict.name not in docker_current_services
{% endraw %}
```

## Docker Swarm

Every virtual machine runs the latest (v1.12) version of Docker. Each host belongs to an Ansible group defined in the Vagrant file. This allows us to target specific types of boxes.

Docker 1.12 is installed via curl as per the docker [release](https://github.com/docker/docker/releases/tag/v1.12.1) page. We only do this if there is no docker systemd service installed on the host.

```yaml
- name: Check existence of docker
  stat: path=/usr/lib/systemd/system/docker.service
  register: install_result

- name: Install docker 1.12
  shell: >
    curl -fsSL https://get.docker.com/ | sh
  when: not install_result.stat.exists

- name: Start docker service
  service: name=docker state=started enabled=true
```

Before we run the swarm init command we establish whether there is an existing swarm running on this machine. Currently this is done by talking directly to the docker daemon. We know its running as docker as we have previously started it.

```yaml
{% raw %}
- name: Check the docker status
  no_log: true
  shell: >
    echo -e "GET /info HTTP/1.0\r\n" | nc -U /var/run/docker.sock | tail -n +6 | python -m json.tool
  register: docker_result

- set_fact:
    docker_info: "{{ docker_result.stdout | from_json }}"
{% endraw %}
```

### Swarm manager(s)

Initially we start a swarm manager on the first **manager** host

```yaml
{% raw %}
- name: Configure primary swarm manager
  hosts: managers[0]
  become: yes
  become_user: root
  roles:
    - role: ansible.secure-docker-daemon
      dds_host: "{{ vagrant_primary_manager_ip }}"
      dds_server_cert_path: /etc/default/docker
      dds_restart_docker: no

  tasks:
    - name: "Starting primary swarm manager"
      shell: >
        docker swarm init --advertise-addr {{ vagrant_primary_manager_ip }}
      register: init_result
      when: docker_info.Swarm.LocalNodeState != "active"

{% endraw %}
```

Each other manager initialises using the manager token described below

### Swarm tokens

Next we register facts for the manager and worker swarm tokens that are needed when we initialise the other hosts.

```yaml
{% raw %}
- name: "Retrieve manager token"
  shell: >
    docker swarm join-token manager --quiet
  register: manager_token_result

- set_fact:
    manager_token: "{{ manager_token_result.stdout }}"

- name: "Retrieve worker token"
  shell: >
    docker swarm join-token worker --quiet
  register: worker_token_result

- set_fact:
    worker_token: "{{ worker_token_result.stdout }}"
{% endraw %}
```

### Swarm workers

Each worker host connects to the primary manager instance using the worker token.

```yaml
{% raw %}
- hosts: workers
  become: yes
  become_user: root
  tasks:
    - name: "Starting swarm workers"
      shell: >
        docker swarm join \
          --token {{ hostvars['manager1']['worker_token'] }} \
          {{ vagrant_primary_manager_ip }}:{{ swarm_bind_port }}
      register: init_result
      when: docker_info.Swarm.LocalNodeState != "active"
{% endraw %}
```

### Swarm node labels

Sometimes we want to restrict where Docker services can run. The cleanest way of doing this with our current setup is to associate the Ansible groups with the Docker nodes.

```yaml
{% raw %}
- hosts: all
  serial: 1
  become: yes
  become_user: root
  tasks:
    - name: "Label nodes"
      shell: >
        docker node update --label-add {{ item }}=true {{ inventory_hostname }}
      when: "item != 'all_groups'"
      with_items:
        - "{{ group_names }}"
      delegate_to: "{{ groups['managers'][0] }}"
      tags:
        - label
{% endraw %}
```

Node updates should be performed on a manager node. In the above example we run a shell command on all hosts. Since ```docker node update``` needs to be run against a manager node we delegate to the primary manager. We iterate over all the hosts groups ```{% raw %}{{ group_names }}{% endraw %}``` and add a boolean label for each host. This means that we can later restrict services to a named group using ```--constraint 'node.labels.xxxx == true'```

```ruby
TASK [Label nodes] *************************************************************
task path: /Users/jamesdmorgan/Projects/vagrant-ansible-docker-swarm/ansible/swarm.yml:122

skipping: [worker2] => (item=all_groups)  => {"changed": false, "item": "all_groups", "skip_reason": "Conditional check failed", "skipped": true}

changed: [worker2 -> None] => (item=workers) => {"changed": true, "cmd": "docker node update --label-add workers=true worker2", "delta": "0:00:00.014551", "end": "2016-08-30 10:41:57.119466", "item": "workers", "rc": 0, "start": "2016-08-30 10:41:57.104915", "stderr": "", "stdout": "worker2", "stdout_lines": ["worker2"], "warnings": []}
```

We skip over the group **all_groups** as it doesn't add any useful semantics.


### Swarm status

After all the nodes have been added and labeled the status of the swarm is output

```ruby
TASK [debug] *******************************************************************
task path: /Users/jamesdmorgan/Projects/vagrant-ansible-docker-swarm/ansible/swarm.yml:146
ok: [manager1] => {
    "docker_swarm_info.Swarm": {
        "Cluster": {
            "CreatedAt": "2016-08-30T09:41:51.32899677Z",
            "ID": "2m72mdv7c8aslr8vpmis615k4",
            "Spec": {
                "CAConfig": {
                    "NodeCertExpiry": 7776000000000000
                },
                "Dispatcher": {
                    "HeartbeatPeriod": 5000000000
                },
                "Name": "default",
                "Orchestration": {
                    "TaskHistoryRetentionLimit": 5
                },
                "Raft": {
                    "ElectionTick": 3,
                    "HeartbeatTick": 1,
                    "LogEntriesForSlowFollowers": 500,
                    "SnapshotInterval": 10000
                },
                "TaskDefaults": {}
            },
            "UpdatedAt": "2016-08-30T09:41:51.368332853Z",
            "Version": {
                "Index": 11
            }
        },
        "ControlAvailable": true,
        "Error": "",
        "LocalNodeState": "active",
        "Managers": 3,
        "NodeAddr": "192.168.77.21",
        "NodeID": "3d5nu66a7nuyjqauz3imkinwz",
        "Nodes": 6,
        "RemoteManagers": [
            {
                "Addr": "192.168.77.22:2377",
                "NodeID": "dv4fyiq4mbriyqrace841cybd"
            },
            {
                "Addr": "192.168.77.23:2377",
                "NodeID": "0cf1851b78xocoymd3kwvqa1p"
            },
            {
                "Addr": "192.168.77.21:2377",
                "NodeID": "3d5nu66a7nuyjqauz3imkinwz"
            }
        ]
    }
}

PLAY [managers[0]] ***********************
```

This shows that there are 3 manager nodes and 6 nodes in total.

### Docker networks

In order for our services to be able to communicate with each over across the different nodes we need to add an [overlay](https://docs.docker.com/engine/userguide/networking/get-started-overlay/) network. As the system gets more complicated its likely we will want to add more to segment the different layers of the system, split frontend traffic from backend etc.

```yaml
{% raw %}
- name: Create overlay networks
  shell: >
    docker network create -d overlay {{ item }}
  when: item not in docker_current_networks
  with_items:
    - appnet
{% endraw %}
```