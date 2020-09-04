+++
title = "Netboot Pxe"
author = ["Benoit Joly"]
date = 2020-09-03T22:46:47-04:00
lastmod = 2020-09-03T21:13:52-04:00
tags = ["100DaysToOffload"]
categories = ["tech"]
draft = false
+++

## Where are my USB drives? {#where-are-my-usb-drives}

Have you been there too?

I seems to never find my spare USB drives when I need them. This week, I wanted to re-install my laptop but could not find an unused USB drive to put the ISO on.

This is when I found PXE and NetBoot.xyz image.


## What is PXE? {#what-is-pxe}

PXE stands for Preboot eXecution Environment. It's a way to distribute boot images over network.

It uses a combination of TFTP (Tiny File Transfer Protocol) for transferring images and DHCP for discovery.

It enables to boot most machines over network.


## netboot.xyz {#netboot-dot-xyz}

While searching how to setup PXE, I've discovered [netboot.xyz](#netboot-dot-xyz). If you boot one of their images, you will get a menu which allows you to select live images, install images, rescue images.

Here is what it looks like:

{{< figure src="/ox-hugo/menu.png" >}}

And selecting network installs shows this:

{{< figure src="/ox-hugo/install-images.png" >}}

Select an image, it will download and boot it.


## TFTP server {#tftp-server}

Self hosting TFTP is relatively easy if you have a WRT router, PfSense/OpenSense, FreeNAS. It's not much more difficult to install on any Linux server.

In my case, I have the choice between:

1.  DD-WRT :: not the best since it's hiding behind my network in a NAT network.
2.  OpnSense firewall :: Would work but has limited space
3.  other servers :: more complex than other solutions
4.  FreeNAS :: ended up choosing my NAS, where it's primary role is to serve files.

The setup is straight forward:

1.  create a folder where you can store the netboot images
2.  Enable the TFTP service
3.  configure TFTP to use the folder created as it's main directory
4.  store the netboot.xyz images: netboot.xyz.kpxe for regular boot, and netboot.xyz.efi for EFI booting.

If you want to verify your setup, just download a TFTP client (for example, atftp), connect to the TFTP server and _get netboot.xyz.kpxe_. This should download the images served by your TFTP server.


## DHCP {#dhcp}

Once you have TFTP working, the last piece missing is the DHCP configuration.

What happens is your DHCP server, which most PC get an IP from, can give additional information, like the TFTP / images to use during PXE network booting.

This seems to be limited to some routers, but most FOSS firewall or routers allow you to configure PXE.

On my OpnSense Firewall, I need to:

1.  enable network booting
2.  set the TFTP server to my FreeNAS server IP
3.  define the default image with netboot.xyz.kpxe and the EFI 32/64 with netboot.xyz.efi


## Booting up {#booting-up}

Now, to boot using PXE on any PC or even in VirtualBox, you need to select network in the boot menu. After getting an IP address, you should boot the netboot.xyz image.


## Thoughts {#thoughts}

I've missed writing on my blog... Life, vacation, making major backyard refactor put me away from the keyboard for some time.

Special congrats to folks who made it to the 100 posts. This is quite an achievement.

I don't think I'll get this anytime soon, but I plan to continue posting regularly.

---
This is day 14 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>.

<!--more-->
