---
layout: post
title: OSX Development Environment
description: "Automating the setup of a Mac for development"
modified: 2016-08-14
category: osx
tags: [osx,ansible,brew]
---

<section>
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->


## All change

On the 21st July I left Openbet a place that I have both learnt a great deal and a place that I should have probably left a couple of years ago. The last time my laptop running Ubuntu died I decided to automate the process of setting up the next one.

I am in a similar position now in that I will have a new machine. Most likely a Mac. I use Macs at home so I thought I ought to automate the setup of a new machine and apply it to my current one.

> [https://github.com/jamesdmorgan/mac-dev-playbook](https://github.com/jamesdmorgan/mac-dev-playbook)

## Ansible

I have been using Ansible a lot over the last couple of years and love it. I created a fairly basic playbook when I was provisioning my Ubuntu boxes. Googling around, a handful of projects and shared Ansible roles handle many parts of the process including brew / brew cask for osx. Everytime I install something I try and ensure its added to this project.

The main inspiration for the approach to take was from [Jeff Geerling](https://github.com/geerlingguy). I forked his [mac-dev-playbook](https://github.com/geerlingguy/mac-dev-playbook) project and adapted for my [needs](https://github.com/jamesdmorgan/mac-dev-playbook).

## Dotfiles

As part of this process my dotfiles have been combined from my Openbet ones, [Guy's](https://github.com/geerlingguy/dotfiles) and [Mathias Bynens](https://github.com/mathiasbynens/dotfiles)

As part of the forked project Guy has conveniently written a role to install them.

The dotfiles include .osx config settings

## Vim

I have added a role to install and configure vim. It installs pathogen and bundles for things like nerdtree etc.

## Sublime

Sublime is installed and configured. A handful of the packages that are installed are

- Package control
- SideBarEnhancements
- GitGutter

For the full list see [github](https://github.com/jamesdmorgan/mac-dev-playbook/blob/master/roles/sublimetext/defaults/main.yml)

## Installation

See the [readme](https://github.com/jamesdmorgan/mac-dev-playbook#installation) for more info





