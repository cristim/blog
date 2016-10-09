---
author: mcristi
comments: true
date: 2009-05-18 21:20:31+00:00
layout: post
link: https://mcristi.wordpress.com/2009/05/19/new-hardware/
slug: new-hardware
title: New hardware
wordpress_id: 12
categories:
- Avatt
- bios
- coreboot
- flashrom
tags:
- Avatt
- bios
- coreboot
- flashrom
---

Hello,

After my last post I did a little hardware shopping. I bought a motherboard [well-supported by coreboot](http://www.coreboot.org/ASUS_M2V-MX_SE) with socketed [DIP8](http://en.wikipedia.org/w/index.php?title=DIP-8) BIOS flash that's both easy to replace and supports large SPI flash chips, namely an [Asus M2V-MX-SE](http://www.asus.com/product.aspx?P_ID=rO7Bu9kq3D25tXHl&content=specifications). Also, I bought an AMD Opteron CPU and some RAM, and used some old components for the rest of the system.

[Carl-Daniel Hailfinger](http://hailfinger.org), the main [flashrom](http://coreboot.org/flashrom) developer provided me with some large (4MBytes) [SST25VF032B](http://www.sst.com/products.xhtml/serial_flash/25/3.0V/SST25VF032B) flash chips whch provide enough space so that [AVATT](http://coreboot.org/avatt) could fit inside. Some coleagues from my university's electronics faculty soldered one of the chips on a PCB, something like this: ![my flash chips](http://mcristi.files.wordpress.com/2009/05/img_2890.jpg?w=300)

I hacked a few days on getting coreboot run properly on this new motherboard of mine, but unfortunately a couple of days ago I mistakenly burned my main BIOS image with some garbage. I tried to recover it using the latest proprietary BIOS image, which obviously failed after the next reboot, and rendered the board unbootable until [Rudolf Marek](http://assembler.cz/) - another coreboot developer who owns the same motherboard as me - provides me with a working flash, hopefully soon...

Until then, I'll be working on a new project of mine which aims to build a  GUI utility that helps porting coreboot to new motherboards.

I'll post updates as soon as my new flash arrives or my new project advances notably.

Stay tuned,

Cristi
