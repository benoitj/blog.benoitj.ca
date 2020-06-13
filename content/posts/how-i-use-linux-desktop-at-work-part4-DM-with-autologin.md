+++
title = "Display Manager With Autologin - How I Use Linux Desktop at Work - Part4"
author = ["Benoit Joly"]
date = 2020-06-12T21:19:25-04:00
lastmod = 2020-06-13T08:33:04-04:00
tags = ["Linux", "coding", "tools", "vm", "100DaysToOffload"]
categories = ["tech"]
draft = false
+++

Part 4 of the series _How I use Linux desktop at work_ where I describe how I'm building a new Vagrant configuration to use for coding on Windows Host systems.

This post describes the learning process to setup LightDM with autologin.

As with previous posts, I will go over the research and manual implementation first, then go over how to automate it.

In part 3, I described how I automated the build of the VirtualBox guest additions using vagrant. Part 4 will improve our current box by swapping the display manager to LightDM and setup autologin.

Previous posts:

1.  [part1](https://blog.benoitj.ca/2020-05-29-how-i-use-linux-desktop-at-work-part1-basic-setup/) - Basic setup
2.  [part2](https://blog.benoitj.ca/2020-06-09-how-i-use-linux-desktop-at-work-part2-wm/) - X, and WM
3.  [part3](https://blog.benoitj.ca/2020-06-10-how-i-use-linux-desktop-at-work-part3-guest-additions/) - VirtualBox guest additions

In the post, I'll cover:

1.  Rationale
2.  Installing lightDM, a greeter
3.  Configure autologin
4.  Automating the whole thing
5.  Thoughts and next steps

Steps 1-4 are describing the manual process while step 5 describe how I automated this in the Vagrantfile.

**Note**: You can find the sources created during this post in my github [devbox-arch](https://github.com/benoitj/devbox-arch/tree/part4) repository.


## Rationale {#rationale}

I want to setup autologin since I want to get in the guest linux box ASAP without unecessary login.

I only want to do this on a vagrant box under certain conditions:

1.  This vagrant box has a virtual NAT network enabled and no port forwarded except ssh using ssh keys setup by vagrant
2.  This box is secured by my host OS credentials. I cannot get in this box without unlocking my Host desktop.


## Installing lightDM, a greeter, and XSession {#installing-lightdm-a-greeter-and-xsession}

I really like the quality, somewhat simplicity and features that lightDM gives.

The setup I usually put in place with lightDM:

1.  lightDM itself
2.  the gtk greeter. There are better greeter, but this one is simple, available everywhere
3.  a xsession that starts my ~/.xinitrc

In arch, this means installing lightdm, lightdm-gtk-greeter, and xinit-xsession (AUR)

For the first two, its simple:

```bash
pacman -S lightdm lightdm-gtk-greeter
```

xinit-xsession is part of AUR, so it's a bit more involved as you have to build it from source.

```bash
# some required tools
pacman -S git fakeroot

# now clone the AUR git repository
git clone https://aur.archlinux.org/xinit-xsession.git
cd xinit-xsession
makepkg -sic --noconfirm
```

Now time to enable lightdm:

```bash
systemctl enable lightdm
systemctl start lightdm
```

You'll now see the lightdm greeter with the _vagrant_ user pre-selected.

If you want to login, the password is _vagrant_.

Note that you can select xinitrc as a session in the session selection at the top right of the screen.


## Configure autologin {#configure-autologin}

Autoconfig is configured in lightDM main configuration _/etc/lightdm/lightdm.conf_.

You must set the following two properties to get autologin with xinit session enabled:

```configuration
[seat:*]
autologin-user=vagrant
autologin-session=xinitrc
```

Note: you have to uncomment the two properties under the existing seat configuration.

You can restart lightdm to see it in action:

```bash
systemctl restart lightdm
```

You also need to add the autologin group and add user vagrant to this group:

```bash
groupadd -r autologin
usermod -a -G autologin vagrant
```


## Automating the whole thing {#automating-the-whole-thing}

All changes were made to the 2-core.sh script.

The interesting part is how to install xinit-xsession and then setup lightdm for autologin.

Installing xinit-xsession using AUR comes to one command:

```bash
cd /tmp
    && sudo -u vagrant git clone https://aur.archlinux.org/xinit-xsession.git && \
    cd xinit-xsession && \
    sudo -u vagrant git checkout 7cae213844b && \
    sudo -u vagrant makepkg -sic --noconfirm
```

Note: makepkg needs to run as a regular user, so you need to clone, and call makepkg using vagrant.

Enabling autologin can be done using sed:

```bash
sed -i -e 's/# *autologin-user=.*/autologin-user=vagrant/' /etc/lightdm/lightdm.conf
sed -i -e 's/# *autologin-session=.*/autologin-session=xinitrc/' /etc/lightdm/lightdm.conf
```

The sed's -i option enables in-place editing which change the file specified.

The last part, creating a .xinitrc that calls openbox:

```bash
cat <<EOT > ~vagrant/.xinitrc
#!/bin/bash
exec /usr/bin/openbox
EOT

chown vagrant:vagrant ~vagrant/.xinitrc
chmod +x ~vagrant/.xinitrc
```

Here, the file is created as root, and then ownership is transferred to the vagrant user.


## Thoughts and next steps {#thoughts-and-next-steps}

I hope I am able to demonstrate how easy it is to provision a Linux development VM using vagrant and some unix scripts.

The hardest part is done, getting a foundation to install development tools and configuration.

In next post, I'll configure this VM using chezmoi. This will allow me to bring my dotfiles and installation scripts.

---

When I started this blog, I was hoping it would be useful to others. Now I see that it may be as useful to me. It makes me thinks of my goals, think about improving, it makes me learn.

I hope this series inspires people to shape tools to meet their needs.

_This is day 6 of my #100DaysToOffload. You can read more about the challenge here: <https://100daystooffload.com>._

\#+hugo more
