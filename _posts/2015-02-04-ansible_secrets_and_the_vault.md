---
layout: post
title: Ansible Vault & GPG
description: "Sharing secret information using Ansible and GPG"
modified: 2015-01-14
category: continuous-delivery
tags: [ansible, vault, gpg, security]
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

There are numerous ways to encrypt and share sensitive information. Projects such as [BlackBox](https://github.com/StackExchange/blackbox) , [Pass](http://www.passwordstore.org/) and [Ansible Vault](http://docs.ansible.com/playbooks_vault.html) do similar things. BlackBox has the added benefit that multiple people can access the sensitive information as long as they are on the same keyring using GPG

As Ansible is used a the main configuration management tool and will be to the tool that's manipulating the secret / sensitive information it makes sense to try and leverage vault. Thanks to John Knight the following [post](https://btl.gs/2014/09/01/using-ansible-vault-and-gpg-to-secure-critical-infrastructure-in-public-github-repositories/) covers off how to combine Ansible, GPG & Git to provide a similar but more integrated solution than Blackbox.

This post will build on that information and add extra background reading.

### Useful resources

* [news.ycombinator.com](https://news.ycombinator.com/item?id=8264496)<br/>
* [Ansible Vault & GPG](https://btl.gs/2014/09/01/using-ansible-vault-and-gpg-to-secure-critical-infrastructure-in-public-github-repositories/)<br/>
* [Ansible cookbook 2014](http://ansiblecookbook.com/html/en.html#how-do-i-store-private-data-in-git-for-ansible)<br/>
* [GNU Privacy](https://www.gnupg.org/gph/en/manual.html)<br/>

## Secrets

The rationale for this work is to build a docker container that acts as a Jenkins Slave. It needs to have sensitive information stored inside in order to access git, svn and git etc. This information needs to be stored in the project encrypted

* cvspass auth information
* svn auth information
* builder private key

All this information should be stored inside a vault encrypted file in Ansible. Im using the following structure.

{% highlight yaml %}
vars/
└── vault
    ├── cvs.yml
    ├── keys.yml
    └── subversion.yml
{% endhighlight %}

{% highlight yaml %}
> ansible-vault encrypt vars/vault/* --vault-password-file ./vault-password.txt
{% endhighlight %}

As the ansible playbook is run passing the vault password the content can be simply copied

{% highlight bash %}
sudo docker run \
    -h centos-slave \
    --name=building base_image \
    ansible-playbook \
        --vault-password-file /srv/ansible/vault-password.txt \
        -c local \
        -s -v /srv/ansible/site.yml
{% endhighlight %}

{% highlight yaml %}
- name: Added builder private key from vault to .ssh
  copy:
    content: "{{ vault_builder_id_rsa }}"
    dest: "{{ builder_dir }}/.ssh/id_rsa"
    owner: "{{ builder_user }}"
    group: "{{ builder_group }}"
    mode: '0600'
{% endhighlight %}

## Makefile

The docker container is built using Make which wraps up all the docker commands and allows the password to be unencrypted.

{% highlight bash %}
{% raw %}
SHELL := /bin/bash
img:
    @echo -------------------------------------------------------
    @echo Cleaning up previous containers
    @echo -------------------------------------------------------

    CONTAINER="$(shell sudo docker ps -a  | grep building | awk '{ print $$1 }')" ; \
        if [[ -n "$$CONTAINER" ]]; then \
            sudo docker rm $$CONTAINER; \
        fi

    # Decrypt the vault password
    gpg --batch --yes --decrypt-files vault-password.txt.gpg;

    mvn -U clean package

    @echo  -------------------------------------------------------
    @echo  Building docker container via Ansible
    @echo  -------------------------------------------------------

    sudo docker build -t base_key_ansible target/build/jenkins_slave
    sudo docker run \
        -h centos-slave \
        --name=building base_key_ansible \
        ansible-playbook \
            --vault-password-file /srv/ansible/vault-password.txt \
            -c local \
            -s -v /srv/ansible/site.yml

    @echo  -------------------------------------------------------
    @echo  Built docker container
    @echo  -------------------------------------------------------
    sudo docker commit building centos-slave

    @echo  -------------------------------------------------------
    @echo  Cleaning up
    @echo  -------------------------------------------------------
    sudo docker rm building
    sudo docker ps -a
    sudo docker images
    sudo docker rmi base_key_ansible

    mvn clean

    rm vault-password.txt

    @echo  -------------------------------------------------------
    @echo  Complete
    @echo  -------------------------------------------------------
{% endraw %}
{% endhighlight %}