---
layout: post
title: Unix Cheat Sheet
description: "Cheat Sheet for Unix and Bash"
tags: [unix,bash]
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

## Useful commands

##### Recursively remove CVS $header$ headers
{% highlight css %}
for file in `find . -type f | xargs`
do
    sed '/# $Header/d' $file > $file.new
    mv $file.new $file
done
{% endhighlight %}


##### IP Whitelist apache virtual hosts
The below code will only allow localhost access to the yum repos, example taken from [blog.lysender.com](http://blog.lysender.com/2013/02/white-listing-ip-addresses-for-your-apache-virtual-hosts/)

{% highlight xml %}
<Directory "/var/www/html/yum">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order deny,allow
    Allow from 127.0.0.1
    Allow from xxx.xxx.xxx.xxx
    Allow from xxx.xxx.xxx.xxx
    Deny from all
</Directory>
{% endhighlight %}