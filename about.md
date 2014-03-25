---
layout: page
permalink: /about/
title: About this site
tagline: Road to CID
cdtags: [about, Jekyll, theme, responsive]
modified: 9-9-2013
image:
  feature: branches-feature.jpg
---
## Purposes of this blog

The blog is currently an experiment with Jekyll & Github pages. If it were the late 90's I would have an under constuction logo... I aim to post information which I hope will be useful to others 
involved in Continuous Integration, Maven, Sonar, Jenkins etc. 

## A little bit of history

Over the last year and a half I have been introducing CI into my company. The goal was relatively simple, if ambitious. We wanted to achieve
one-click deployment of the entire system. Whilst this has been achieved the scope for improvement is vast. There are many unsolved challenges.

<br/>
To give some scale to the endeavour virtually all the applications had no unit tests, were built from scratch every time a release was cut. The applications
were mostly written in TCL which has a very small if non-existent community and no framework support. Most of the applications are versioned in CVS.

<br/>
When researching CI/CD for this project most examples I found cater for modern languages / frameworks and source control. Each non-standard component has added extra
complexity and uncovered many obsure issues. It has taken lots of digging.


## Goal of this site
My aim is to share some of the knowledge I have learnt working with "legacy" applications and systems and to plan out how I go about open sourcing some of the code I have written.
<br/>
As my work has progressed the solution has matured and in some places is more elegant than many answers posted on stack overflow etc. So I thought it was time I shared them.
<br/>
Maven is usually associated with Java projects but its quite capable of providing the lifecycle for any type of application.

## Continuous integration pipeline
In implementing the following pipeline I have had to created numerous connectors and even built a xUnit framework. I plan on detailing things i've learnt in the hope
that it could prove useful.

* Jenkins with standard Maven jobs
* Maven with custom Ant plugin for packaging non-standard application
* Maven lifecycle invokes custom TCL xUnit mocking framework for unit testing.
* Publishes artefacts to Sonar for analysis and Artifactory
* Aggregate Maven project consumes artefacts and creates publishes full system
* Jenkins deploys system to VMWare Virtual Machines

The majority of the work has been on the unit testing / mocking framework and the Maven plugin. The plugin contains an Ant library of macros which handle

* Building and packaging artefacts
* Unpacking plugin dependencies via Java mojo
* SCM tasks for CVS / SVN / Git
* CI tasks for talking to Jenkins via CLI
  * Cloning jobs
  * Building custom jobs
* Building release notes and communicating with Confluence

The plugin uses both the Ant contrib packages and xmltask additions all pulled in nicely as Maven dependencies.

<br/>
The plugin also provides a menu system for common automated tasks such as preparing a project for a release and defect workflows. I have generally tried to
automate our companies workflow. This has become more important as the number of steps needed to setup the Jenkins jobs increases. Many steps are usually ignore by
developers.
