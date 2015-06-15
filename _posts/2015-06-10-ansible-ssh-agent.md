---
layout: post
title: SSH Forwarding for Ansible
description: "SSH Forwarding and Bastion configuration for Ansible"
modified: 2015-06-10
category: ansible
tags: [ansible]
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


## Resources

* [SSH Agent Forwarding](https://developer.github.com/guides/using-ssh-agent-forwarding/)<br/>
* [SSH Through Bastion] (https://10mi2.wordpress.com/2015/01/14/using-ssh-through-a-bastion-host-transparently)<br/>

## Ansible environment variables to forward SSH info through bastion
{% highlight bash %}
# When using ssh-agent need to forward SSH settings for Bastion machine
export ANSIBLE_TRANSPORT="ssh"
export ANSIBLE_SSH_ARGS="-o ForwardAgent=yes"
{% endhighlight %}

## SSH Config settings for Bastion proxy

{% highlight bash %}
Host abc*
    User jmorgan

Host abc
    Hostname abc-bastion.cloudapp.net

Host abc* !abc-bastion.cloudapp.net
    ProxyCommand ssh abc -W %h:%p
{% endhighlight %}

## Automatically setup ssh-agent as part of bash session
{% gist 45ef2be24d9ff13b33ba %}





