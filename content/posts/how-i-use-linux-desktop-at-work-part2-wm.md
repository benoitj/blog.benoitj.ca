+++
title = "How I use Linux desktop at work - Part2 - Window manager and GUI"
author = ["Benoit Joly"]
date = 2020-06-09T16:31:24-04:00
lastmod = 2020-06-09T16:34:34-04:00
tags = ["Linux", "coding", "tools", "vm", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

Part 2 of the series _How I use Linux desktop at work_

Just to recap, this series is describing how I leverage vagrant and VirtualBox to provision development VM on my work PCs. I'm also taking the opportunity to build them from scratch with the tools I use at home.

In the post, we go over:

1.  creating a basic arch VM
2.  configure VM
3.  install X related packages
4.  Install one Window Manager (lxde)

The end result is a versioned configuration that can provision a basic arch VM with X and a window manager.

**Note**: You can find the sources created during this post in my github [devbox-arch](https://github.com/benoitj/devbox-arch/tree/part2) repository.


## Pre-requisite {#pre-requisite}

Stuff assumed to be configured before starting and already covered in [part1](https://blog.benoitj.ca/2020-05-29-how-i-use-linux-desktop-at-work-part1-basic-setup/):

1.  VirtualBox
2.  vagrant

Also I make use of [git bash](https://gitforwindows.org/) to version control my configuration and also to run vagrant.

**Note**: Many commands below will run fine on git bash, but not on powershell. Just take this into consideration.


## Creating the basic arch VM {#creating-the-basic-arch-vm}

At this stage, we'll create a bare minimum vagrant file that creates VM based on arch.

I use the official arch vagrant box called archlinux/archlinux.

In my git bash, I type:

```bash
mkdir devbox-arch
cd devbox-arch
vagrant init archlinux/archlinux
```

That's it. Lets boot it up and connect to it:

```bash
vagrant up
vagrant ssh
uname -a
exit
```


## setup a git repository {#setup-a-git-repository}

Might as well create a git repository right away. This will create a safe environment to break things and revert if needed.

```bash
git init
echo ".vagrant/" >> .gitignore
git add Vagrantfile .gitignore
git commit -m "basic arch console config"
```


## configure VM {#configure-vm}

The VM we just created is good enough for console tools, but if we are going to use this for a development VM running X, we need to allocate more resources, setup graphic properties, setup host/guest integration.

Here is my vagrant file, it includes variables to define various properties like memory, CPUs, monitors. My objective is to be able to quickly adapt to the resources available on the host computer I use.

Couple of things to note about the vagrant file below:

1.  ensure that you don't over allocate resources. Don't assign all your CPUs and ram to the VM. If you launch VirtualBox UI, can you see safe limits in the settings menu.
2.  VM customization below starting with vb. are specific to VirtualBox, if you use a different vagrant provider, these options won't work.
3.  Clipboard is setup bidirectional between host and guest
4.  The host folder _C:\Users\\<userid>\src_ (~/src in git bash) is mapped to the guest folder ~/src.
    This is usually the source folder I use to share sources.

<!--listend-->

```ruby
# -*- mode: ruby -*
# vi: set ft=ruby :

NB_MONITORS = "1"
NB_CPUS = 6
MEMORY = "8192"
VRAM = "64"

Vagrant.configure("2") do |config|
  config.vm.box = "archlinux/archlinux"

  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    vb.gui = true
    vb.name = "devbox-arch"

    # resources
    vb.memory = MEMORY
    vb.customize ["modifyvm", :id, "--cpus", NB_CPUS]

    # graphics
    vb.customize ["modifyvm", :id, "--monitorcount", NB_MONITORS]
    vb.customize ["modifyvm", :id, "--vram", VRAM]
    vb.customize ["modifyvm", :id, "--graphicscontroller", "vmsvga"]
    vb.customize ["modifyvm", :id, "--accelerate3d", "off"]

    # Customise audio
    vb.customize ["modifyvm", :id, "--audioout", "on"]
    vb.customize ["modifyvm", :id, "--audio", "dsound"]
    vb.customize ["modifyvm", :id, "--audiocontroller", "hda"]

    vb.customize ['modifyvm', :id, '--clipboard-mode', 'bidirectional']

  end

  # mount my Users\<>\src folder on host to /home/vagrant/src
  config.vm.synced_folder ENV['USERPROFILE'] + "/src", "/home/vagrant/src"
end

```

You may want to destroy your VM (vagrant destroy) and recreate it (vagrant up).


### Setting up additional VM options {#setting-up-additional-vm-options}

You may want to add / change the configuration described above. The vb.customize configurations above use the VBoxManage.exe command from the _C:\Program Files\oracle\virtualbox_ folder.

Call VBoxManage to get more details or you can head over to the [VBoxManage](https://www.virtualbox.org/manual/ch08.html#vboxmanage-modifyvm) documentation.


## configure arch {#configure-arch}

My strategy to provision the configure the OS is to keep the configuration out of the VagrantFile and create a series of shell scripts called by the VagrantFile.

This is done by the provisioner options.

Here is an example calling a specific script as root (by default).

```ruby
config.vm.provision "shell", path: "1-setup-arch.sh"
```

Followed by a vagrant command to run the provision script:

```bash
vagrant provision
```

The config.vm.provision instruction supports inline scripting as well as normal user provisioning.

The setup arch contains instructions to:

1.  setup timezone and locale
2.  configure pacman to use a regional mirror
3.  update the package list and update with any new package versions


## Getting X environment working {#getting-x-environment-working}

Same as with arch config, I'm using a script to setup X and related software.

```ruby
config.vm.provision "shell", path: "2-core.sh"
```

Again followed by:

```bash
vagrant provision
```

Some objectives for the core provisioning script is:

1.  install X, it's drivers, a window manager
2.  setup audio
3.  install virtualbox guest additions with X support

Now you should see a GUI login prompt with the vagrant user selected. You can enter the super secret password (ie: vagrant).

**Warning**: The vagrant account is safe as long as you don't weaken the existing setup (like: enable services using password authentication).


### Time to speed up {#time-to-speed-up}

I'm running out of time, and as I previously posted, I prefer something working to something perfect.

Here are the things I'll be improving and describing in future posts:

1.  setup an auto logger to get in the vagrant account instead of a session manager
2.  add additional core apps like vim.
3.  replace lxde with WM of choice (dwm) with related tools/configuration

So in a nutshell, the 2-core.sh script will evolve.


## Things that could go wrong {#things-that-could-go-wrong}

Got two issues while building this configuration:

1.  pacman update fails due to bad mirror.

    This is why I've been selecting the first mirror that works for me

2.  guest OS resolution does not adjust with the VM window size. I wasted quite some time trying to find why.

    This issue is due to incompatibility with your VirtualBox and the install guest additions.
    you have two choices here:

    1.  install the exact same version as the virtualbox-guest-utils package. Does not always work.
    2.  build the guest additions from the VM iso by selecting "insert guest addition" in the VirtualBox menus. I will create a new post how I've done this.


## where to find sources {#where-to-find-sources}

This is becoming my default setup for both home and work.

You will find this particular configuration in my git repository: [devbox-arch.](https://github.com/benoitj/devbox-arch/tree/part2)

If you look at the master branch, you'll most definitively find something different as this configuration will evolve over time.


## What is coming next {#what-is-coming-next}

Some topics for the near future:

1.  New post to cover how to get window resizing by installing/compiling the guest additions
2.  Using chezmoi to manage dotfiles and tools installation

---

I hope this series is of some use to others and inspire people to use tools that suits their needs.

_This is day 4 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._
