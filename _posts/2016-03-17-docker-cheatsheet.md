---
layout: post
title: Docker Cheatsheet
description: "Useful notes, observations and commands for docker"
modified: 2016-03-17
category: docker
tags: [docker]
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


## Docker socket

{% highlight bash%}
echo -e "GET /info HTTP/1.0\r\n" | nc -U /var/run/docker.sock
HTTP/1.0 200 OK
Content-Type: application/json
Server: Docker/1.9.1 (linux)
Date: Thu, 17 Mar 2016 15:54:57 GMT
Content-Length: 1444

{"ID":"VUTL:QKSZ:5RYC:W33H:4EAT:7ECE:TQ47:3H56:GCJY:3GJW:RPFX:5CCN","Containers":13,"Images":104,"Driver":"aufs","DriverStatus"
...
{% endhighlight %}

Communicating between containers is possible by talking to the socker server via the socket on the host

* [Sending signals between containers](http://blog.dixo.net/2015/02/sending-signals-from-one-docker-container-to-another/)
* [Docker Remote API v1.22](https://docs.docker.com/engine/reference/api/docker_remote_api_v1.22/)