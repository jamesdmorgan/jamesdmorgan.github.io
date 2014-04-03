---
layout: post
title: Custom Maven plugins
description: "Building a plugin with a little hindsight"
modified: 2014-04-03
category: continuous-integration
tags: [maven, java, ant, groovy]
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

# Different approach
If I had to write a Maven plugin and I wasn't constrained by needing to incorporate Ant I would stick to Java & Groovy. Maybe even just groovy.

# Reasons for building a Maven plugin

Originally our applications were built from source every time they were release. I initially trialled Ant & Ivy for building and
deploying the built applications to Artifactory. Since many of the Java applications were using Maven I moved away from Ivy. At the time
Ivy also had issues with classifiers, which helped change the approach.
<br/><br/>
All non-standard parts of the build were now handled by a collection of Ant tasks. This was necessary since I was building applications that
has no Maven support. The Ant scripts were released to a shared NFS partition. They weren't properly versioned and really needed to be consumed as a Maven dependency.


# Building on top of Ant

Converting to a Maven Ant plugin wasn't overly difficult as the online documentation was fairly good. The complications arose with the fact that I wanted
to include other dependencies within the plugin and have them downloaded when the plugin was invoked. These dependencies were things like the unit testing framework I wrote
and tools to communicate with Jenkins & Confluence.

## Limitations of Ant Mojos
At the time of creating Ant based mojos the functionality available was slightly more limited than the annotations available in the Java mojos.
One of the major limitations other than the verbose mojo definitions was accessing the Maven project properties from within the Ant execution. This is a real problem
if you need to access properties that can't be defined in advance. The <b>antrun</b> plugin does this for you by pre-populating ant properties.

## Contrib, Macros & Scriptdefs

Having Ant tasks depend on other Ant tasks gets complicated very quickly. Whilst Ant is imperative I appreciate its not designed to be used like a programming language.
<br/><br/>
Using the contrib package simplified a great deal of the tasks with the use of if, then, else conditions. As many of the scripts
were organised as a nested collection of xml files includes and calling tasks conflicted.
<br/><br/>
Moving to using macros and scriptdefs added the next level of control. If you are restricted with using Ant I would opt for this approach.
The next evolution was adding the <b>&lt;groovy&gt;</b> task.

# Java & Groovy mojos
The first Java mojo came about as a need to unpack the plugins dependencies. It utilises Tim Moore's [Mojo Executor]( https://github.com/TimMoore/mojo-executor).
This allowed me to pass the list of artifacts as a configuration through and execute the Maven Dependency plugin Unpack mojo.

# Migrating Ant tasks to Groovy

I'm now in the process of moving tasks from Ant into Groovy & Java. Groovy provides a very easy way of calling ant tasks. Currently the libraries to communicate with
Artifactory are in Java and the glue is Groovy. I would quite like to move all Ant tasks to this framework as its more flexible, clearer.
<br/><br/>
Ant provides quite a rich library of tasks that I've still found very useful, moving, copying and deleting files etc.
<br/>
<br/>
There are a couple of major limitations using Maven/Plexus & Ant which i'll cover in another post relating to symbolic links and file permissions.

# Splitting up the plugin
Currently the plugin covers a range of tasks
<br/>

* Testing TCL applications
* Packaging non-standard applications
* Building manifests
* Automating workflows, Source control management
* Jenkins job creation and configuration for Sonar
* Artifactory REST support for tagging and searching
* Confluence integration for release notes and wiki pages

<br/>
<br/>
My plan is to break apart the plugin
<br/>

* Integrate the TCL unit testing framework into surefire
* Create a plugin dedicated to Jenkins / Artifactory functionality

Whilst the majority of the code is encapsulated and separates business logic from functionality there would still need to be an amount of work before anything could be open sourced.
