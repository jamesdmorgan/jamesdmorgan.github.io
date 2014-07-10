---
layout: post
title: Database Cheat Sheet
description: "Cheat Sheet for database tips and commands"
tags: [informix, postgresql, sql, databases]
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

## Informix

###### Who is connected to a database (change wildcard)
{% highlight css %}
select
  sysdatabases.name database, -- Database Name
  syssessions.username,       -- User Name
  syssessions.hostname,       -- Workstation
  syslocks.owner sid          -- Informix Session ID
from
  syslocks,
  sysdatabases ,
  syssessions
where
  syslocks.tabname = "sysdatabases"       -- Find locks on sysdatabases
and syslocks.rowidlk = sysdatabases.rowid -- Join rowid to database
and syslocks.owner   = syssessions.sid    -- Session ID to get user info
and sysdatabases.name like '%'
order by 1
{% endhighlight %}

## PostgreSQL