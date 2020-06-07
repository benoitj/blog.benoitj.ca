+++
title = "How I use Linux desktop at work - Part2 - Windown manager and GUI"
author = ["Benoit Joly"]
date = 2020-06-05T11:31:24-04:00
lastmod = 2020-06-06T11:30:11-04:00
tags = ["Linux", "coding", "tools", "vm", "100DaysToOffload"]
categories = ["tech"]
draft = true
+++

Part 2 of the series _How I use Linux desktop at work_

Just to recap, this series is describing how I leverage vagrant and virtualbox to provision development VM on my work PCs. I'm also taking the opportunity to build them from scratch with the tools I use at home.

In the post, we go over:

1.  creating a basic arch VM
2.  configure memory/cpu and disk allocation
3.  install X related packages
4.  Install one Window Manager (TBD: Mate?, xfce?, Cinnamon? )


## Creating the basic arch VM {#creating-the-basic-arch-vm}

This will be an arch based vagrant VM, and I'll use the official arch vagrant box:

```bash
mkdir devbox-arch
cd devbox-arch
vagrant init archlinux/archlinux
```

That's it. Lets try it:

```bash
vagrant up
vagrant ssh
uname -a
```


## configure memory/cpu and disk allocation {#configure-memory-cpu-and-disk-allocation}

This is good enough for console, but if we are going to use this for a development VM running X, we need to allocate more resources, setup graphic properties, setup host/guest integration.

here is what the vagrant file looks like now:

```ruby
TODO=123
```
