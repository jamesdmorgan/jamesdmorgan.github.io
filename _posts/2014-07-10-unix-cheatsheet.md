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

###### Recursively remove CVS $header$ headers
{% highlight css %}
for file in `find . -type f | xargs`
do
    sed '/# $Header/d' $file > $file.new
    mv $file.new $file
done
{% endhighlight %}