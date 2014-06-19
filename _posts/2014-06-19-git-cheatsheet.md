---
layout: post
title: Git Cheat Sheet
description: "Cheat Sheet for Git & Gitolite"
tags: [sample post, code, highlighting]
link: http://stackoverflow.com/questions/501407/is-there-a-git-merge-dry-run-option
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

## Git

###### Emulate CVS merge without committing or [fast forward](http://stackoverflow.com/questions/501407/is-there-a-git-merge-dry-run-option)
{% highlight css %}
git merge --no-commit --no-ff $BRANCH
{% endhighlight %}

## Gitolite

###### Add permissions to adhoc repo
{% highlight css %}
ssh git@git.openbet perms -h
{% endhighlight %}

{% highlight css %}
Usage:  ssh git@host perms -l <repo>
        ssh git@host perms <repo> - <rolename> <username>
        ssh git@host perms <repo> + <rolename> <username>

List or set permissions for user-created ("wild") repo.  The first usage shown
will list the current contents of the permissions file.  The other two will
change permissions, adding or removing a user from a role.

Examples:
    ssh git@host perms foo + READERS user1
    ssh git@host perms foo + READERS user2
    ssh git@host perms foo + READERS user3

----
There is also a batch mode useful for scripting and bulk loading.  Do not
combine this with the +/- mode above.  This mode also accepts an optional "-c"
flag to create the repo if it does not already exist (assuming $GL_USER has
permissions to create it).

Examples:
    cat copy-of-backed-up-gl-perms | ssh git@host perms <repo>
    cat copy-of-backed-up-gl-perms | ssh git@host perms -c <repo>
{% endhighlight %}