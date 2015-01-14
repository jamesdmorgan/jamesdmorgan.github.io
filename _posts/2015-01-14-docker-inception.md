---
layout: post
title: Docker Inception
description: "Building docker containers to build docker and AMI images"
modified: 2015-01-14
category: continuous-delivery
tags: [ansible, packer, docker]
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

## Ansible building docker to build docker containers with ansible?

Why would I want to do this?<br/>

We need a standardised environment that has Ansible and Packer installed in order to build Docker & Amazon AMI images. <br/>

This environment may as well be a Docker container, which we may as well build using Ansible as it allows us to keep the Dockerfile simple.<br/>

Ansible can provide us a base Docker image available in the [registry](https://registry.hub.docker.com/repos/ansible/)<br/>

## Useful resources

* [Gotchas with Dockerfiles](http://kimh.github.io/blog/en/docker/gotchas-in-writing-dockerfile-en/)<br/>


## Ansible bootstrapped Dockerfile

The Dockerfile for this is very simple and can be extended to add Packer and other utilities needed to build our images

{% highlight bash %}
# Latest version of centos
FROM centos:centos7
MAINTAINER Toshio Kuratomi <tkuratomi@ansible.com>
RUN yum clean all && \
    yum -y install epel-release && \
    yum -y install PyYAML python-jinja2 python-httplib2 python-keyczar python-paramiko python-setuptools git python-pip
RUN mkdir /etc/ansible/
RUN echo '[local]\nlocalhost\n' > /etc/ansible/hosts
RUN pip install ansible
{% endhighlight %}

A similar approach is descriped in this post [installing-and-building-docker-with-ansible](http://www.ansible.com/2014/02/12/installing-and-building-docker-with-ansible)

## Calling local Ansible playbook and installing Packer

{% gist 458fbeb5afa8c627380e %}

![](/images/icons/github-32px.png) [Github example](https://github.com/jamesdmorgan/ansible-docker-packer)

