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

### Who is connected to a database (change wildcard)
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

### What tables have a foreign key to a tables PK
{% highlight css %}
select
  e.tabname,
  g1.colname
from
  systables      a,
  sysconstraints b,
  sysreferences  c,
  sysconstraints d,
  systables      e,
  sysindexes     f,
  syscolumns     g1
where
  a.tabname      = 'TABLENAME'
and a.tabid      = b.tabid
and b.constrtype ='P'
and b.constrid   = c.primary
and b.tabid      = c.ptabid
and c.constrid   = d.constrid
and d.tabid      = e.tabid
and e.tabid      = f.tabid
and f.idxname    = d.idxname
and f.tabid      = g1.tabid
and abs(f.part1) = g1.colno
{% endhighlight %}

### Adding a new dbspace (chunk)

#### Useful resources
* [Informix knowledge base](http://www-01.ibm.com/support/knowledgecenter/SSGU8G_12.1.0/com.ibm.adref.doc/ids_adr_0464.htm)


Create the space
{% highlight css %}
touch /opt/informix_11.70/data/datadbs_02
{% endhighlight %}

{% highlight css %}
chmod 660 /opt/informix_11.70/data/datadbs_02
{% endhighlight %}

Add the chunk
{% highlight css %}
onspaces \
  -a datadbs \
  -p /opt/informix_11.70/data/datadbs_02 \
  -o 0
  -s 2048000 # size of chunk in kb
{% endhighlight %}

Check the space is available and force a level 0 backup
{% highlight css %}
onstat -d
ontape -s -L 0 -d > /dev/null
{% endhighlight %}

{% highlight css %}
onstat -m
{% endhighlight %}

The following is a useful query to check the amount of free space
{% highlight css %}
select
  name[1,8] dbspace,
  sum(chksize) Pages_size,
  sum(chksize) - sum(nfree) Pages_used,
  sum(nfree) Pages_free,
  round ((sum(nfree))/(sum(chksize))*100,2) percent_free
from
  sysdbspaces d, syschunks c
where
  d.dbsnum=c.dbsnum
group by 1
order by 1;
{% endhighlight %}

## PostgreSQL