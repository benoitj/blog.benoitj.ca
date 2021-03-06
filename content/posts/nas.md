+++
title = "Home data storage"
author = ["Benoit Joly"]
date = 2020-09-05T20:26:48-04:00
lastmod = 2020-09-06T10:25:00-04:00
tags = ["selfhosting", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

## A need to own your data {#a-need-to-own-your-data}

The workday today ended with my friend and work buddy telling me it's about time he own and centralize his data (for him, wife and kids).

I've had servers since the 90s, so obviously I had solutions for him :P

Like mine, he and his family use a variety of devices at home, phones, laptop, desktop.

The two obvious choice to self host your data are:

1.  a Network-Attached Storage (NAS) server
2.  a decentralized network of computers replicating your data (aka: Syncthing)

While Syncthing is great for smaller data sets, it requires to share the same data set at least on 2 machines. When you have terabytes of data, it becomes not practical.

So this post is about running your NAS. A NAS offers multiple ways to access your data like NFS, Samba, SSH/SFTP, WebDav and more. How to setup your NAS is beyond this post, and will possibly be the topic of a future post.


## Made for you or home made? {#made-for-you-or-home-made}

There are two main track for running your own NAS at home:

1.  Getting a store bought NAS appliance (synology, Qnap, Netgear, and others)
2.  Building your own


### Store bought NAS {#store-bought-nas}

Here are some reasons to choose a store bought NAS appliance:

1.  Simplicity, beside disks, you get the box, motherboard, CPU, ram, SATA controller, and software sorted for you. Just pick a brand and the number of drives supported, and you're done with choices
2.  Fast to get started, put drives in, boot and setup
3.  Usually small footprint

Some minuses:

1.  Usually expensive, especially if you want to put more than 2 drives
2.  Not flexible, software is usually proprietary and the computer is lower power unless you pay big $$$
3.  Data at risk when the hardware dies. You may want to setup redundancy which on these NAS is implemented by proprietary setup. If the NAS dies, you may have to find the same branch, possibly the same model/family to recover your data.


### Build your own {#build-your-own}

Some pros:

1.  Make your own, you make all the choices
2.  Usually cheaper for higher capacity NAS than a store bought appliance
3.  easier to put back online if one of the key components dies since it will be based on software RAID
4.  more software options including software RAID
5.  For Geeks, more fun :D

Obvious cons:

1.  more decisions to make
2.  more complex
3.  Willing to learn, read, research, break things, especially if you have limited knowledge of running servers. If you want stuff done for you, building your own may not be the right fit.

My family would find it obvious that I suggested my friend to build his own :D

Some things you should plan for:

1.  a big box, with enough room for your drives and at least 1/2 more space to add drives as your needs change.
2.  OS Drive. You usually want to store the OS on a separate disk. I really small SSD is a good choice.
3.  disk controller
    1.  onboard: already included. slower, less quality, and usually more limited in number of ports. rarely more than 4/6 SATA ports
    2.  SAS/SATA card: much faster, robust. you can find cards used in server cheap on eBay. A 8 ports LSI SAS 2008 card cost less than 100$.
4.  Storage Drives. Two paths SAS or SATA. You may want to vary brand or batch to reduce the risk for multiple disk to fail
    1.  SAS, robust, but expensive. you can buy used drives though
    2.  SATA: easily obtainable used
5.  a decent CPU/MB/Memory, possibly with ECC Ram. You don't need to buy new, but decent hardware. Some people run out of raspberry pi, but this limits performance due to requiring USB drives which are slower (USB) than SATA/SAS drives on a SAS or SATA bus. Make sure you have PCIe expansion to host

My setup:

1.  Cheap Rosewill 4U server box
2.  i3 / supermicro / 16 gig ECC
3.  many 3T WD RED drives bought at different time
4.  a small 128 gig SSD
5.  a controller LSi 9210-8i running a P20 IT pass-through firmware needed to run FreeNAS / ZFS and bypass the hardware RAID on the LSI card.


## Redundant array of inexpensive drives (RAID) {#redundant-array-of-inexpensive-drives--raid}

Lets start with some definitions.

RAID is a way to describes different arrays of drives to achieve increased capacity, or redundancy, or both.

Here are some important ones:

RAID 0
: a stripe of 1 or more drives. You basically have multiple drives acting as one big drive (the sum of all drives). This adds capacity to make a bigger "logical" drive

RAID 1
: 2 disk or more setup as mirrors. 2 drives is the minimum, but some systems allow to add more drives to the mirror. The obvious advantage is redundancy, a lesser known is faster read (you have more disks to read from); The main disadvantages, cost and write has to be done on multiple drives.

RAID 5 or RAIDZ (ZFS)
: Act as a stripe, but address redundancy by adding one drive to allow a parity bit. Don't be mistaken, the additional parity bits are not stored on one single drive, but spread on all drives. ZFS support additional parity bits for added redundancy. Some pros: capacity, lower cost than a mirror. Cons: Slower read than a RAID 1.

RAID 10
: combine RAID 1 (mirror) to RAID 0 (stripe). In this configuration, mirrors are concatenated into a stripe for added capacity. This combines both redundancy, capacity and read performance, but increase cost.

My favorite RAID, RAID 10. Cost / Gig is so low these days that it's less of a problem these days.


## Volume Manager + File systems {#volume-manager-plus-file-systems}

I've been playing with two main setup:

-   Linux Volume Manager (LVM) to give me software RAID, and EXT3/4 for file system
-   ZFS for both volume manager and file system

I won't into the details of each, both are great and my favorite is ZFS. It's not more complex than LVM / EXT3/4 and it gives additional features like snapshots, and easy replication.


## Software {#software}

Here you have plenty of options:

1.  your own custom Linux + other software
2.  OpenMediaVault (OMV), a linux based NAS OS
3.  FreeNAS, a FreeBSD based NAS OS
4.  Other proprietary NAS software

I've tried my own, OMV and FreeNAS. With ZFS support, solid feature set, and stability, FreeNAS is my go to OS for my NAS.

To install it, you have to boot the installation image, install the OS on your OS drive, then configure your ZFS pool and data volumes.


## Backup {#backup}

When you plan for a NAS you must also consider backup:

1.  local
2.  remote

You need both local and remote backups to be resilient to lightning, flood, someone stealing, or just your own mistakes.


### Local backup {#local-backup}

For local backup, you can setup ZFS snapshots. ZFS snapshots are taking a "picture" of your files at a given time. Some of my ZFS volumes have daily snapshots, some have hourly snapshots and I usually keep them for one month.

ZFS Snapshot + RAID 10 gives me a relatively fast recovery time in case of issues or failures.


### Remote backup {#remote-backup}

You can either buy hosting for your backup, which is relatively slow, could be costly, and could have transfer limits. If you have of data to backup, both speed and possibly cost will become an issue.

The other option is to host a low power machine with enough drives to get your data. I currently have an old core 2 duo running at my parents place. I run daily ZFS snapshot replication which copies my snapshots to the remote server every day. The worst I can loose is one day of data.

Now my friend needs a remote backup solution, so we both agreed to host each other's backup. For this, he'll send me a large drive, and I'll send him one too (and decommission my old server at my parents place). Since I want to keep my data to myself, I will most probably use Borg backup to backup and encrypt my latest ZFS snapshot, and send this over to my friend's NAS.

You also may have a friend or family member who can host an old server, and maybe share the cost.


## Other services {#other-services}

Once your NAS is setup, you can add other distribution services, like NextCloud, plex/emby, airsonic (music), or calibre (books). You can run all of these on FreeNAS if you have enough RAM/CPU.


## Thought {#thought}

Your NAS does not have to be that powerful, if all you have is documents and some pictures, a raspberry pi or similar is enough. If your NAS has multiple users and server lots of (large) files, it may require a bigger setup like the one describes in this post.

Setting up your own NAS is possibly the first step to own your data.

---

_This is day 15 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

<!--more-->
