+++
title = "How I Use Linux Desktop at Work - Part 1 - Basic setup"
author = ["Benoit Joly"]
date = 2020-05-29T11:31:24-04:00
lastmod = 2020-06-10T11:10:49-04:00
tags = ["Linux", "coding", "tools", "vm", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

Most coders is large companies are facing the same situation: We develop for Linux servers, but we are running on Windows or MacOS boxes.

Things are working _mostly_ the same at best, at worst, it's broken.

We need a solution and [Vagrant](https://www.vagrantup.com) is a quick way to provision a Linux VM on your local desktop.

This will be a series of posts starting with these 4 topics:

1.  Basic setup
2.  Getting basic X setup
3.  Make it your own. Dotfiles and development environments
4.  Variants using boxes from Linux OS


## Why Vagrant {#why-vagrant}

For those who don't know, [Vagrant by HashiCorp](https://www.vagrantup.com/) is a VM provisioning tool targeted at spawning software development boxes. I've used Vagrant for the last 3-4 years to quickly get a Linux development VM running on my Windows Desktop.

Vagrant support various VM/Hypervisor platforms like libvirt/KVM (Linux), MS HyperV (Win), [Oracle VM VirtualBox](https://www.virtualbox.org) (both), VMWare

[Vagrant](https://www.vagrantup.com) primary file is a VagrantFile which describes how the VM must be configured. It goes from which OS to use, how many memory/cpu to use, disk size, shared drive between Host and Guest, and much more.

There are couple of reasons to use this setup to configure a Linux development environment:

1.  Quick and reproducible
2.  Can be shared with colleagues and friends
3.  Isolate the environment from your host OS. Can become project specific setup.
4.  Develop on the same OS as your target OS
5.  You can use vagrant to setup a set of VMs that works together (things like a Kubernetes cluster)


## Virtualization {#virtualization}

There are couple of [choices](https://www.vagrantup.com/docs/providers) of virtualization platform:

1.  Virtualbox
2.  VMWare
3.  Docker
4.  HyperV
5.  WSL (beta)

In most cases, Virtualbox is the best solution due to it's Guest Additions integration and USB2.0/3.0 support.

In short, why others are lesser capable IMHO:

-   VMWare: cost something. You can get away by running only the player, but it's not as capable than the regular product or Virtualbox
-   Docker: on Windows, docker runs on a HyperViser, I find this adds a layer of complexity to the VM approach. I also use docker daily, and running docker in docker can work, but it needs special configuration
-   HyperV: The configuration is quite similar to Virtualbox. Capable configuration UI, and quick fast execution. Where it fall short compared to VirtualBox is it uses [RDP](https://en.wikipedia.org/wiki/Remote%5FDesktop%5FProtocol) instead of a seamless integration with the Host we get with Virtualbox. If it does not bother you, then HyperV is a valid option, and maybe a faster one.
-   WSL: First, support for WSL in Vagrant is beta. It's also only text based (I know you can go around it). Not really giving what you need compared to the alternatives
-   WSL2: There are some tickets open to support WSL2. This may be a viable option in the future. Maybe this will bridge the gap between HyperV and Virtualbox with regards to seamless integration with the host OS. Too early to tell.


## Basic setup {#basic-setup}

The aim of this first part is to get a prompt on the Guest OS of choice, in this case Arch, but it could be any other Linuxes.


### Enable virtualization {#enable-virtualization}

To make things fast, we need to use Virtualization implemented using Hardware virtualization capabilities of your PC. In theory, software based virtualization could work, but it's far from being usable if you want to run X and a window manager (ie: way too slow).

Most PC these days support Hardware virtualization (even my old x220 from 2012), but it may not be enabled.

You can enable virtualization by going to your BIOS and enabling hardware virtualization. It can have various names, VT-x, Virtualization, or something similar. If not enabled, Virtualbox wont be able to give you hardware virtualization. If this is on your work PC, you may need your IT department to enable this if they password protected your BIOS.


### Install Virtualbox {#install-virtualbox}

Install [Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads). In my case, I install the VirtualBox for Windows Host.

You can optionally install the Oracle VM VirtualBox Extension Pack which gives you host bridging with USB 2.0/3.0 devices. This is required if you want to pass-through a USB 2.0 or 3.0 device connected to your host directly to your VM.


### Vagrant {#vagrant}

Vagrant requires newer version of Powershell to be able to install. Windows 10 has the right version and I'll assume you are on Windows 10 since other versions are EOL.

Head over [Vagrant by HashiCorp](https://www.vagrantup.com/) and download vagrant.

Installation should be trivial.


### Vagrant Box selection {#vagrant-box-selection}

Time for box selection. A vagrant box is a pre-configured bare VM that you can download and start. It's the VM equivalent to a docker image.

Head over to [Discover Vagrant Boxes - Vagrant Cloud](https://app.vagrantup.com/boxes/search) page to search for boxes.

Like any community contributed software, take special care to select a box you can trust.

**Disclaimer**: The box I'm going to select here is based on my own evaluation. I'm not flawless, and also things could change over time (ie: what is safe today may not be in the future). You should do your own assessment. You've been warned, don't blame me if this breaks your system.

I would recommend to take boxes from reputable sources, or build you own (which is way beyond the scope of this series).

To test the setup, I'll just select a small VM for alpine: _generic/alpine38_


### VagrantFile {#vagrantfile}

Vagrant is configured in a similar way to docker, with a VagrantFile used to "build" your VM.

To create the simplest VagrantFile for a new setup, open a cmd prompt or powershell and type:

```powershell
mkdir test-vm
cd test-vm
vagrant init generic/alpine38
```

BTW, Vagrant is written in ruby and the VagrantFile is a ruby file.

Here is the equivalent of your Vagrantfile:

```ruby
Vagrant.configure("2") do |config|
  config.vm.box = "generic/alpine38"
end
```


### Booting the box {#booting-the-box}

Now that your configuration file is created, run the following command in a powershell in the created directory:

```powershell
vagrant up
```

In the end of the process, you should have a VM provisioned and started.


### Connect to your box {#connect-to-your-box}

From the same vagrant folder, you can connect to your box using ssh with keys generated for your:

```powershell
vagrant ssh
```

You are now connected to your box using SSH.

You can now disconnect from SSH (ie: exit or ^d)


### Sharing files using default /vagrant folder {#sharing-files-using-default-vagrant-folder}

You may have noted that vagrant mounted _/vagrant_ to your local vagrant folder. In your box, you can share files from that folder. There are ways to mount additional folders and we'll go over this in a later post.


### Shutting down {#shutting-down}

Another useful command, is you can tell vagrant to shutdown the VM using:

```powershell
vagrant halt
```


### Destroying your box {#destroying-your-box}

Since this is just a test box, you may want to delete the VM:

```powershell
vagrant destroy
```


## What I get out of this {#what-i-get-out-of-this}

As you can see, Vagrant helps you setup a minimal Linux OS running in a VM in just a few steps.

This is way more efficient than installing from ISO and going through the installation.

Coming next will be part 2, addressing basic X setup as well as tweaking the VM configuration using the same VagrantFile.

_This is day 3 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
