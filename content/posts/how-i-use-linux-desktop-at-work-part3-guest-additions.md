+++
title = "How I use Linux desktop at work - Part3 - Guest additions"
author = ["Benoit Joly"]
date = 2020-06-10T16:31:24-04:00
lastmod = 2020-06-10T01:01:52-04:00
tags = ["Linux", "coding", "tools", "vm", "100DaysToOffload"]
categories = ["tech"]
draft = true
+++

Part 3 of the series _How I use Linux desktop at work_

Just to recap, this series is describing how I leverage vagrant and VirtualBox to provision development VM on my work PCs. I'm also taking the opportunity to build them from scratch with the tools I use at home.

Previous posts:

1.  [part1](https://blog.benoitj.ca/2020-05-29-how-i-use-linux-desktop-at-work-part1-basic-setup/) - Basic setup
2.  [part2](https://blog.benoitj.ca/2020-06-09-how-i-use-linux-desktop-at-work-part2-wm/) - X, and WM

In the post, I'll cover:

1.  reasons why I ended up compiling the guest additions
2.  installing required tools
3.  mounting drive
4.  building guest additions
5.  automating the whole thing

The end result will make the X guest screen resize to the host window size.

**Note**: You can find the sources created during this post in my github [devbox-arch](https://github.com/benoitj/devbox-arch/tree/part3) repository.


## Reasons {#reasons}

During part 2, I spent most of my time trying to get X guest screen to resize automatically. I tried many things like, installing the dkms package, downgrading my virtualbox to the exact version of guest additions available in arch. All of this time without positive results.

Let me know if you know how to solve it without building guest additions.


## Installing tools {#installing-tools}

guest additions needs which, gcc, perl, linux-headers packages in order to be compiled.

you can do so by running these commands.

```bash
pacman -S which perl gcc linux-headers
```


## Mounting drive {#mounting-drive}

From the devices menu, select the "insert guest additions CD..." which is the equivalent of inserting the CD in the virtualdrive

Now mount this iso to the /mnt mount point

```bash
mount /dev/sr0 /mnt
```


## Building guest additions {#building-guest-additions}

Now you just need to go to the /mnt path and run the installer:

```bash
cd /mnt
./VBoxLinuxAdditions.run
```

Accept the installation.

Then reboot and voila!


## Automating the whole thing {#automating-the-whole-thing}

Now, we do not want to do this manually everytime, lets automate this.

This comes in two parts:

1.  binding the iso to the DVD drive
2.  mounting and building


### Binding the iso to the DVD drive {#binding-the-iso-to-the-dvd-drive}

Now automating binding of the iso is fairly easy with vagrant. You just have to use the modifyvm to create a CD drive and tell to "put" the iso file in the drive:

```ruby
vb.customize ['storageattach', :id, '--storagectl', 'IDE Controller', '--device', 1, '--port', 1, '--type', 'dvddrive', '--medium', 'C:\Program Files\Oracle\Virtualbox\VBoxGuestAdditions.iso']
```


### Mounting and building {#mounting-and-building}

Like I described in part 2, we will update the "2-core.sh" script to build the guest additions.

```bash
mount /dev/sr0 /mnt && cd /mnt && ./VBoxLinuxAdditions.run -- --force
umount -f /mnt
```


### Rebooting {#rebooting}

I use the vagrant-reload plugin to reboot while provisioning.

Once installed, you can reboot the VM with:

```ruby
config.vm.provision :reload
```


## Value of all this {#value-of-all-this}

At this stage, with just a single vagrant up command, I can automatically provision a linux VM with X configured and a basic window manager.

This will serve as a foundation to my work/home dev boxes.


## What is coming next {#what-is-coming-next}

Some topics for the near future:

1.  Swap the X session manager to an auto-logger
2.  Configure dwm/st/sxhkd/dmenu using chezmoi dotfiles management to provision everything
3.  Coding tools installation

This may just be one post, we'll see.

---

I hope this series is of some use to others and inspire people to use tools that suits their needs.

_This is day 5 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._
