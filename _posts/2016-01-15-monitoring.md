---
layout: post
title: Monitoring
description: "Improved system monitoring ideas"
modified: 2016-01-15
category: monitoring
tags: [nagios, incinga2, collectd, statsd, influxdb, grafana]
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


## Overview

Our current monitoring solution is based almost entirely around Nagios and custom NRPE scripts.
There are pros and cons of using Nagios, though I am not going into that here. There are many [comparisons](http://phillbarber.blogspot.co.uk/2015/03/nagios-vs-sensu-vs-icinga2.html)
on the internet.
<br/>
We need a lightweight, scalable solution that is easy to configure and manage and provides mechanism for analysing the data and acting on it, not just collecting it. [Redefing monitoring](http://ryanfrantz.com/posts/solving-monitoring/)
<br/>
I believe we also need to build on simply detecting errors to predicting them. We also collect a huge amount of valuable information in the logs. Quite a lot is structured. This information provides invaluable insight into customer behaviour.
<br/>
Monitoring can be broken down in 4 main tasks

* Collection

* Routing

* Visualisation

* Alerting

We have a custom monitoring tool that runs on each client that parses the application logs and formats up messages for the NRPE daemon. This gets picked up by Nagios and routed back to a central server using NSCA.
<br/>
Many customers manually manage their Nagios configuration. This is both unworkable and bad practice. As part of the new customer rollouts and moving to Infrastructure as code (entire stack is managed by Ansible) I converted the existing Nagios & NRPE configuration to Ansible roles. This has simplified the deployment of new servers though suffers from poor visualisation (currently nagiosgraph), performance, alerting.


## Ideas

### Data storage

The system produces a large amount of log data and metrics. We glean small amounts of information from the logs but only scratch the surface of the amount of useful data.
<br/>
It is important that we identify what is important to treat as metrics and what we should do with the rest of the log data. The latter may be less useful for realtime monitoring of the system but could, if collected correctly add insight into customer behaviour and trends. [Discussions](https://discuss.elastic.co/t/elk-vs-grafana-influxdb/1686/6) on application log data vs metrics suggest that the data should be treated differently.

> Grafana + InfluxDB for purely time series metrics (specifically, monitoring applications & servers), and ELK for monitoring/diagnostics against log file sources

ELK (Elastic Search / LogStash & Kibana ):

* Elasticsearch for deep search and data analytics
* Logstash for centralized logging, log enrichment and parsing
* Kibana for powerful and beautiful data visualizations

From reading reviews and comparisons Kibana seems to fall short when it comes to aggregation of building more complex queries. Grafana has [Elasticsearch](http://docs.grafana.org/datasources/elasticsearch/) support so it seems to make sense to use Grafana in place of Kibana.
<br/>
I aim to focus primarily on metrics initially then move on to taming the log files.

### Metrics

Many of our clients have has success with a combination of using CollectD / StatsD on the client, writing to InfluxDB time series database to store the data points and then representing and graphing this information using Grafana. They also leverage the existing NRPE scripts that are used for application monitoring using Icinga2, writing to InfluxDB.

* CollectD could replace the *system* NRPE checks
* CollectD can talk directly to InfluxDB to store the points
* Existing log parsing could be replaced with Logster
* Riemann could optionally sit between CollectD / StatsD to provide contextual alerting, event stream processing
* Icinga2 would replace Nagios to improve the UI and configuration. Icinga can also talk to InfluxDB.
* InfluxDB would be a data source for Grafana
* Icinga2 can pull in graphs from Grafana

### CollectD vs Telegraf?

[Telegraf](https://github.com/influxdata/telegraf/blob/master/README.md) is written by InfluxDB and very new to the scene. It can receive stats from a number of plugins including StatsD and output to a host of services including InfluxDB. There seems [debate](https://news.ycombinator.com/item?id=9746698) on why another tools is needed when CollectD / StatsD already exist but it could given time provide a cleaner / simpler solution.

## Resources

* [CollectD](https://collectd.org/)
* [StatsD in CollectD](https://anomaly.io/statsd-in-collectd/)<br/>
* [Grafana Installation](http://docs.grafana.org/installation/rpm/)<br/>
* [Grafana Docs](http://docs.grafana.org/)<br/>
* [InfluxDB Installation](https://docs.influxdata.com/influxdb/v0.9/introduction/installation/)<br/>
* [Deploying InfluxDB with Ansible](https://influxdb.com/blog/2015/11/19/deploying_influxdb_with_ansible.html)<br/>
* [InfluxDB Ansible playbooks](https://github.com/allen13/influxdb-ansible)<br/>
* [Incinga2 -> InfluxDB -> Grafana](http://blog.wuliwala.net/2015/12/09/icinga2-influxdb-grafana-integration/)<br/>
* [Etsy Logster](https://github.com/etsy/logster)<br/>
* [Riemann -> InfluxDB -> Grafana Slides](http://www.slideshare.net/nickchappell/pdx-devops-graphite-replacement)<br/>
* [CollectD -> Riemann plugin](http://riemann.io/clients.html)<br/>
* [CollectD -> Riemann](https://asylum.madhouse-project.org/blog/2014/12/09/monitoring-setup/)
* [ELK Stack](https://qbox.io/blog/welcome-to-the-elk-stack-elasticsearch-logstash-kibana)

## Deployment and Configuration

This section will be expanded as I start installing and configuring the different components.




