---
author: mcristi
comments: true
date: 2011-06-19 18:23:14+00:00
layout: post
link: https://mcristi.wordpress.com/2011/06/19/preparing-and-ubuntu-image-for-serial-console-on-a-hard-disk-connected-over-usb/
slug: preparing-and-ubuntu-image-for-serial-console-on-a-hard-disk-connected-over-usb
title: Preparing and Ubuntu image for serial console, on a hard-disk connected over
  USB
wordpress_id: 72
---

It's been almost a year since my last post, hopefully I will be able to post more often from now on.

This time I'm making a howto on how to install Ubuntu on a SATA disk-drive while having it connected over USB through an USB2SATA adapter, then how to customize Ubuntu so that all the boot messages and the console are directed to a serial port console.

**What Am I trying to do?**

You might ask yourselves why would you want to do that... Well, I don't know about you, but I needed this in order to prepare my coreboot development environment on a motherboard that I will only access over Serial port or SSH. Now a bit of history... I've been in Berlin for the last three months as part of a business trip, sent by the notorious Finnish mobile phone company that I am working for. While I was at LinuxTag back in May, I finally met the coreboot developers that I've been chatting with on IRC ever since 3 years ago and bought myself a coreboot-supported motherboard (Asrock E350M1) from one of the coreboot developers living in Berlin, Peter 'CareBear\' Stuge. I bought it because I've been planing for quite a while now to build myself a home computer or set-top-box for my TV back at home, and this board seems to be perfect for the job. As a bonus, it is off course running coreboot and quite hacker-friendly.

The coreboot support for this board is still work in progress and although there are a few rough edges, the motherboard is running pretty well, and booting up very fast (under 1 second to the Grub menu). Still, there are a few problems here and there and as a coreboot developer that I like to say I am, although my contributions to coreboot were minor so far, I would like to help getting this board better supported.

The prefered debugging mecanism of coreboot is the serial console because it's relatively easy to initialize and pretty common. Unfortunately this board doesn't provide a console port on the back panel, but it has a header with the required pins somewhere on the PCB.

Yesterday me and Peter spent a lot of time working on this board, trying to build a serial header for it and getting it up to speed for coreboot development. We bought some components and then Peter built a nice serial-to-header adapter that also works ad a NULL-modem serial cable since I didn't have a proper NULL-modem cable.

Then we tried to get an OS running on the board from a SSD drive, but unfortunately the image we had was not properly set up, so we decided to build a new OS installation.

**Hardware Setup**

As I said so far, I have the Asrock motherboard, a serial-to USB adapter and the custom serial header adapter made by Peter. Besides these, I also have a laptop and a portable laptop SATA hard-drive with an USB-to-SATA adapter.

**Software setup**

I chose to do it with Ubuntu because it's easy to set up, quick to install, and pretty nice for development. The hard-disk was connected over USB and I slready had it partitioned, so I only reused the first partition already created there.

I reformatted the first partition to EXT4.


<blockquote>sudo mkfs.ext4 -L rootfs /dev/sdb1</blockquote>


Ubuntu then mounted the first partition to /media/rootfs after double clicking on it.

Installing the base Ubuntu packaged in there. You can replace the architecture to i386 for a 32bit OS, natty with another Ubuntu release, and choose a mirror closer to you.


<blockquote>sudo debootstrap --arch amd64 natty /media/rootfs http://de.archive.ubuntu.com/ubuntu/</blockquote>


After this is done, we can bind-mount some filesystems from the host, preparing for our chroot into the new Ubuntu install.


<blockquote>sudo mount -o bind /dev /media/rootfs/dev

sudo mount -o bind /proc /media/rootfs/proc

sudo mount -o bind /sys /media/rootfs/sys</blockquote>


And finally, chroot


<blockquote>sudo chroot /media/rootfs /bin/bash</blockquote>


Create some config files in the new system


<blockquote>cat << EOF >  /etc/fstab
# device mount type options freq passno
LABEL=root / ext3 defaults,errors=remount-ro 0 1
LABEL=swap none swap sw 0 0
EOF

echo coreboot > /etc/hostname</blockquote>


Set up networking for DHCP


<blockquote>echo -e "auto eth0 \n iface eth0 inet dhcp" >/etc/network/interfaces</blockquote>


Add "restricted universe multiverse" to the line you should have in /etc/apt/sources.list

Install some vital packages


<blockquote>apt-get install linux-image grub-pc</blockquote>


Serial port configuration for Grub

Open /etc/default/grub with an editor.

Comment out


<blockquote>#GRUB_HIDDEN_TIMEOUT=0</blockquote>


Set


<blockquote>GRUB_CMDLINE_LINUX_DEFAULT="console=ttyS0,115200"</blockquote>


Add these two lines


<blockquote>GRUB_TERMINAL=serial
GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"</blockquote>


Then you can update the grub configuration.


<blockquote>update-grub</blockquote>


Install grub on the hard-disk


<blockquote>grub-install /dev/sdb</blockquote>


Configure Linux console on the serial port


<blockquote>cat << EOF >  /etc/init/ttyS0.conf
# ttyS0 - getty
#
# This service maintains a getty on ttyS0 from the point the system is
# started until it is shut down again.

start on stopped rc RUNLEVEL=[2345]
stop on runlevel [!2345]

respawn
exec /sbin/getty -L 115200 ttyS0 vt102
EOF</blockquote>


Set a root pasword


<blockquote>passwd</blockquote>


Exit the chroot, unmount all the directories mounted there, connect the hard-disk and the serial cable to the motherboard and enjoy the new OS over the serial console.

~Cristi
