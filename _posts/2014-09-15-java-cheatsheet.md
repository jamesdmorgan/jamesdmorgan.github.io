---
layout: post
title: Java Cheat Sheet
description: "Cheat Sheet for Java"
tags: [java]
link:
comments: true
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

## System commands and settings

###### Add aes256-cbc to client cipher for SSH connections
When using Jenkins Publish over SSH if the client doesn't use the correct cipher you get the following error

{% highlight css %}
jenkins.plugins.publish_over.BapPublisherException: Failed to connect session for config [XX-XX-XX-XX]. Message [Algorithm negotiation fail]
{% endhighlight %}

This can be corrected by adding Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files for the relevant [JDK](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

{% highlight css %}
> /usr/java/jdk1.7.0_25/jre/lib/security/
-rw-r--r--.  1 root root  2500 Sep 15 13:17 local_policy.jar
-rw-r--r--.  1 root root     0 Jun  6  2013 trusted.libraries
-rw-r--r--.  1 root root  2487 Sep 15 13:17 US_export_policy.jar
{% endhighlight %}


