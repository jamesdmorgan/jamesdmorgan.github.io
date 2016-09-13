---
layout: post
title: Centralised Logging and Data Insight
description: "Centralised logging and insight using ELK (elastic) stack"
modified: 2016-08-30
category: [devops,logging]
tags: [devops,logging,swarm, ansible, elk, elasticsearch, kibana, logstash,rsyslog]
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


## Centralised logging

When you are running a couple of containers its not too difficult to examine a container's logs.

If it follows the [Twelve Factor App](https://12factor.net/) approach to software design the application should be logging to **stdout**

The logs can then be examined using ```docker logs -f container```

As you start to have more containers or use Swarm this approach becomes unworkable very quickly. Especially so in a production environment.

Therefore its important that we start centralising the logging.

Docker introduced log drivers a number of releases ago. These have been extended to **Docker services** in v1.12.

There a number of different log drivers but I am using [rsyslog](http://www.rsyslog.com/) feeding into [Logstash](https://www.elastic.co/products/logstash)

### Docker log driver

The syslog driver is enabled when the service or container is run. In the example below a tag is passed in **Go** format. This will prefix all messages in syslog with the name of the image, container name and ID. The tag is wrapped in raw/endraw to protect the braces.

```yaml
docker service create \
  --name some_service \
  --log-driver syslog \
  --log-opt tag={% raw %}'{{.ImageName}}/{{.Name}}/{{.ID}}'{% endraw %} \
  your/container
```

> Note that when you use the syslog driver you can no longer use ```docker logs``` on the container.

### Ansible role.

The [rsyslog](https://github.com/jamesdmorgan/vagrant-ansible-docker-swarm/tree/master/ansible/roles/rsyslog) role installs the service and adds logstash configuration. This simply forwards messages to the logstash instance.

```
{% raw %}
*.* @@{{ logstash_host }}:{{ logstash_port }}
{% endraw %}
```

If you are running in production you may want to consider adding a **redis** instance between **rsyslog** and **logstash**. This can add fault tolerance if parts of the chain go down for whatever reason. This can be achieved on AWS using a redis elasticache instance.

## ELK / Elastic Stack

In our basic example we are passing logs directly from the **docker** containers via **rsyslog**, through **logstash** and **elasticsearch** finally to be viewed using **kibana**. The example is more to prove connectivity rather than demonstrate many of the features.

An easy way to verify that all the components are working correctly is to scale the containers using the ```docker service scale``` command. Increasing or decreasing the number of containers across the cluster will generate logs an push data through to elasticsearch.

Each component in the ELK stack is run as a container from [logging.yml](https://github.com/jamesdmorgan/vagrant-ansible-docker-swarm/blob/master/ansible/logging.yml) and **rsyslog** is installed on all hosts. The group **elk** is controlled by the Vagrantfile ```ANSIBLE_GROUPS``` variable.


