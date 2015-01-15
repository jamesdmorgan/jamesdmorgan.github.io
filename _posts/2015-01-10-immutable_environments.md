---
layout: post
title: Immutable Environments
description: "Ideas around immutable environments (work in progress)"
modified: 2015-01-10
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

## Overview

The concept of immutable environments is that once a box is built it is never modified. Any change that needs to happen requires a new box to be created and deployed.
<br/><br/>
For this to happen the entire system needs to be under configuration management. This ensures that every component is managed and versioned.
<br/><br/>
Prior to cloud provisioning and configuration management servers were at best built from scripts, at worst manually configured.
<br/><br/>
Configuration management tools such as Puppet, Chef & Ansible have automated the creation and ongoing management of servers to ensure that they are in a known state.
<br/><br/>
Docker and Amazon [AMI](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html) have help bridge the gap between the old approach to managing systems and the new immutable approach.
<br/><br/>
This extends the concept of build once , deploy anywhere that is applied to the continuous integration pipeline and artefact creation and takes it to its logical extreme.
<br/><br/>
Docker containers are created and in theory should move through the test phases into production without changing. Similarly Amazon AMI images can be created locally based on pre-defined AMIs, composed in a similar way to Dockers layered filesystem approach and deployed to Amazon.
<br/><br/>

## Links and references
[Immutable deployments](http://blog.codeship.com/immutable-deployments/) Simple overview on the concepts<br/>
[Immutable Servers]( http://martinfowler.com/bliki/ImmutableServer.html)<br/>
[Amazon EC2 AMI creation with Aminator](http://techblog.netflix.com/2013/03/ami-creation-with-aminator.html)<br/>
[Hootsuite Example](http://code.hootsuite.com/build-test-and-automate-server-image-creation/) Build test and automate server image creation<br/>
[Packer](https://packer.io/) Packer is a tool for creating identical machine images for multiple platforms from a single source configuration.<br/>
[Immutable Systems with Ansible](http://www.ansible.com/blog/immutable-systems)<br/>
[Ansible + Packer](http://blog.codeship.com/packer-ansible/)<br/>
[Ansible + Packer EBS AMI](https://github.com/kelseyhightower/packer-provisioner-ansible-local)<br/>
<br/><br/>

## Issues

The idea of immutable environments is a powerful one, though there are complications that need to be considered. Especially when you are dealing with vast monolithic systems and different deployment approaches through the deployment pipeline.
<br/><br/>
I plan to try and use a pragmatic approach to introducing immutable environments. It will be a while until I see them being deployed to production. Primarily due to the different ways we host our systems. Until servers are provisioned into a cloud (be it a private cloud or something like EC2) this mechanism doesn't work.
<br/><br/>
Similarly there is the problem of the databases. These are many terrabytes in size and hosted on large database servers. We have ongoing projects to start breaking up the monolith. New applications are being built so they maintain their entire stack allowing for easier deployment and separation of concern.
<br/><br/>
I don't see this as an all-or-nothing problem. If we adopt a mixed approach of mutable and immutable environments then it is apparent that they need to be built the same way. This has been my worry with Docker, unless they are provisioned in the same way as production systems you run the risk of introducing errors.
<br/><br/>
Until the monolith is broken up the complexity and scale is too big to transition the entire system through to production. Even if we don't consider the database at this stage.
<br/><br/>

## Current continuous deployment approach

We are currently treating all hosts in the same way. Bare metal boxes, VMWare virtualised environments and Amazon EC2 hosts. This allows us to ensure the same process is used for all environments.
<br/><br/>
Hosts are provisioned using Ansible with applications packaged as RPMs and managed via Yum.
<br/><br/>
One of the downsides of this current approach when dealing with Amazon is that there is execessive bandwidth and CPU being consumed in order to get a base AMI setup.
<br/><br/>
Using the concepts of immutable environments I aim to build the AMI locally and upload a new image. Thus reducing the amount of work that is carried out in California.
<br/><br/>

**Update 15/01/15** From doing more research both Packer.io & Aminator both work on the instance in the cloud instead of allowing local operation. This is a real shame as it means that all the software needs to be available in the cloud via Yum. I will continue investigating this. Options at the moment are<br/>

* Look are reducing cost by building AMIs in layers as opposed to building the stack in one go. This is sensible anyway and the playbooks are split to facilitate this.
* Provision docker containers locally and publish these into either EBS EC2 instances or possibly ElasticBeanstalk. The later seems to have quite a few disadvantages according to this [post](http://stackoverflow.com/questions/25832554/amazon-elastic-beanstalk-vs-ec2-instance-with-docker-containers)

We have developed our Ansible playbooks and roles such that applications can be moved and grouped easily whilst still ensuring they are correctly glued together
<br/><br/>
* Apache configuration on the web tier
* Site management configuration
* Services on the hosts

## Development Ideas

* Leverage standard build process with Ansible
* Build docker containers and AMI images with [packer.io](https://packer.io/) using existing Ansible playbooks
* Build base docker and AMI images via CI to reduce time to compose.
* Deploying to test could involve uploading an AMI and allowing the customer to provision themselves.
* Fig could provide a nice way of simplifying development on docker and building on pre-made containers.

TBC


---
