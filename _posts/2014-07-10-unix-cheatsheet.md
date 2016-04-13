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

The above can be achieved more succinctly using perl. For example simple string replacement in place.

{% highlight bash %}
> perl -pi -w -e 's/jdbc_informix_driver/jdbc_informix_url/g;' $( find . -name "*.yml" )
{% endhighlight %}

This would recursively find all yaml files and run sed expression doing in-place edit.

> -e execute the following line of code.
> -i edit in-place
> -w write warnings
> -p loop

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

##### Convert file from ISO88591 -> UTF-8
{% highlight bash %}
> iconv -f ISO-8859-15 -t UTF-8 iso88591.dat > utf8.dat
{% endhighlight %}

##### Hexdump a single line from a file
{% highlight bash %}
> sed '1944q;d' utf8.dat |hexdump -C

00000000  31 31 30 30 30 32 33 37  30 7c 7c 7c 7c 4f 6c 65  |110002370||||Ole|
00000010  20 47 75 6e 6e 61 72 7c  53 6f 6c 73 6b 6a c3 a6  | Gunnar|Solskj..|
00000020  72 7c 7c 7c 7c 0a                                 |r||||.|
00000026
{% endhighlight %}
