---
author: mcristi
comments: true
date: 2009-04-21 18:13:00+00:00
layout: post
link: https://mcristi.wordpress.com/2009/04/21/avattopenvz/
slug: avattopenvz
title: AVATT/OpenVZ
wordpress_id: 8
categories:
- Avatt
- bios
- coreboot
- OpenVZ
- uclibc
---

Hello,  
  
As I promised in my previous post, the OpenVZ support is getting usable. I have a coreboot ROM image containing a payload consisting of a Linux kernel, the OpenVZ tools vzctl and vzquota, along with uClibc and busybox.  
  
To test this stuff, you need to download the ROM image from [here](http://panzer.utcluj.ro/%7Ealien/coreboot/AVATT/BIOS/OpenVZ/bios.bin) along with the qemu [VGA BIOS](http://panzer.utcluj.ro/%7Ealien/coreboot/AVATT/BIOS/vgabios-cirrus.bin). Put both of them in the same directory (let's say ~/BIOS) and just run qemu like this:  


<blockquote>qemu -L ~/BIOS  -hda /dev/zero  
</blockquote>

You can see the tools are there, but In order for it to be really useful you need a disk image containing an OpenVZ template. I haven't created such an Image yet, but stay tuned, soon i'll post a link to such an image.  
  
Cristi
