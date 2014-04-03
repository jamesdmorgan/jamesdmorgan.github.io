---
layout: post
title: Building a pipeline
description: "Quick overview of the kind of CI tasks i've been involved in"
modified: 2014-03-25
category: continuous-integration
tags: [intro, maven, jenkins, ci]
image:
  feature: texture-feature-05.jpg
  credit: Texture Lovers
  creditlink: http://texturelovers.com
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

# A little bit of history

Over the last year and a half I have been introducing CI into my company. The goal was relatively simple, if ambitious. We wanted to achieve
one-click deployment of the entire system. Whilst this has been achieved the scope for improvement is vast. There are many unsolved challenges.

<br/>
To give some scale to the endeavour virtually all the applications had no unit tests, were built from scratch every time a release was cut. The applications
were mostly written in TCL which has a very small if non-existent community and no framework support. Most of the applications are versioned in CVS.

<br/>
When researching CI/CD for this project most examples I found cater for modern languages / frameworks and source control. Each non-standard component has added extra
complexity and uncovered many obsure issues. It has taken lots of digging. Having to think about CVS support as well as git & svn complicates matters



## Goal of this site
My aim is to share some of the knowledge I have learnt working with "legacy" applications and systems and to plan out how I go about open sourcing some of the code I have written.
<br/>
Maven is usually associated with Java projects but its quite capable of providing the lifecycle for any type of application.

## Continuous integration pipeline
In implementing the following pipeline I have had to created numerous connectors and built a xUnit framework. I aim to open source this in the coming months.

* Jenkins with standard Maven jobs
* Maven invokes custom plugin for packaging non-standard applications
* Maven lifecycle invokes custom TCL xUnit mocking framework for unit testing.
* Built artefacts are published to Artifactory & Sonar for analysis
* Aggregate Maven project consumes artefacts and creates full build ready for deployment
* Jenkins deploys system to Virtual Machines via SSH Publish & custom scripts.

The majority of the work has been on the unit testing / mocking framework and the Maven plugin. The plugin contains an Ant library of macros which handle

* Building and packaging artefacts
* Unpacking plugin dependencies via Java mojo
* SCM tasks for CVS / SVN / Git
* CI tasks for talking to Jenkins via CLI
  * Cloning jobs
  * Building custom jobs
* Building release notes and communicating with Confluence
* REST communication with Artifactory

The plugin uses both the Ant contrib packages, groovy and xmltask additions all pulled in nicely as Maven dependencies.

<br/>
The plugin also provides a menu system for common automated tasks such as preparing a project for a release and defect workflows. I have generally tried to
automate our companies workflow. This has become more important as the number of steps needed to setup the Jenkins jobs increases. Many steps are usually ignore by
developers.

