---
author: mcristi
comments: true
date: 2009-03-24 17:18:00+00:00
layout: post
link: https://mcristi.wordpress.com/2009/03/24/some-news-about-the-avatt-bios/
slug: some-news-about-the-avatt-bios
title: Some news about the AVATT BIOS
wordpress_id: 7
categories:
- Avatt
- bios
- coreboot
- kvm
- OpenVZ
- uclibc
---

Hello,  
  
It's been a long time since my last post... many happened in the meanwhile, but I almost forgot about this blog.  
  
This time I'll talk again about my progress on the [AVATT](http://coreboot.org/avatt) project, at whichI worked ever since last summer's GSoC. I got it built with [KVM](http://kvm.qumranet.com/) support (using buildrom), and I achieved starting an [OpenBSD ISO image inside it](http://panzer.utcluj.ro/%7Ealien/coreboot/AVATT/screenshots/running/2008-09-01-233342.png). Soon after the boot, the image crashes with a strange error message, caused by missing [Thread Local Storage(TLS)](http://en.wikipedia.org/wiki/Thread-local_storage) support from uClibc. The image also contains busybox and ncurses, and its compressed size is just a bit under 2MBytes.  
  
Since the uClibc developers haven't yet developed TLS support (and think I'm not yet ready to do it myself), I decided to move on to another virtualization software, namely [OpenVZ](http://openvz.org/). To ease my work - which previously suffered from buildrom's way of compiling the coreboot payload, that is not a cross-compiling toolchain so everything was made to compile using hackish compiler flags - I also decided to switch to a real toolchain, namely Buildroot. This eases a lot the process of compiling a Linux userland payload, making a lot easier the lives of those who want to make something similar to [LBDistro.](http://sourceforge.net/projects/lbdistro/)  
  
So, now I'm working on porting AVATT to OpenVZ. I currently have a ROM image containing a kernel patched with OpenVZ support, and I'm working on the vzctl port to buildroot. See the AVATT wiki page for more, or maybe you can try it yourself, building it from [source](http://repo.or.cz/w/avatt.git).  
  
Stay tuned...there's more to come soon!  
  
Cristi  

