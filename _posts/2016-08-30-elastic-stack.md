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

The [rsyslog](https://github.com/jamesdmorgan/vagrant-ansible-docker-swarm/tree/master/ansible/roles/rsyslog) role installs the service and adds logstash configuration. This simple forwards messages to the logstash instance.

```
{% raw %}
*.* @@{{ logstash_host }}:{{ logstash_port }}
{% endraw %}
```

## ELK / Elastic Stack

TBC


