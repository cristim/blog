---
author: mcristi
comments: true
date: 2009-06-15 21:54:37+00:00
layout: post
link: https://mcristi.wordpress.com/2009/06/16/my-impressions-about-fedora-11-as-compared-with-ubuntu/
slug: my-impressions-about-fedora-11-as-compared-with-ubuntu
title: My impressions about Fedora 11, as compared to Ubuntu
wordpress_id: 34
---

Hello,

After I presented a slide show at the last saturday's [Fedora Launch Party](http://forum.fedoraproject.ro/gallery/displayimage.php?pid=1808&fullsize=1) event, I decided to give Fedora 11 a try for a few days, and switched from Ubuntu 9.04, and by now it seems it will be for a long time.

My first impression about it was when it told me some bad news, it found out that my disk drive is having a bad sector, right from the install medium. Ubuntu didn't show this kind of info at all.

I'm glad that my garbled fonts are gone, but I was a bit badly surprised that they didn't customize Nautilus at all, and by default it behaves just like Windows 95's Explorer.

The system is moving quite well, compared to Ubuntu. The newer kernel, EXT4 and 64bit made a huge difference performance-wise, the system seems quite stable, and Evolution seems not to leak any more the way it did in Ubuntu.

Their new fingerprint-based feature behaves as badly as I managed to get fprint work In Ubuntu(maybe the problem is because of my hardware or my fingers you never know...), just that it is better integrated with the system, so I turned it off after a few swipes :)

They have nice KMS support, but unfortunatelly I'm not using console mode (GNU screen is a nice replacement).  By the way, the screen configuration in Ubuntu rocked, and should be installed by default in here too, IMHO.

I was expecting to see a better display manager tray applet in gnome, but hopefully that will come in a later release, and that the dual display feature works at lease as good as in Ubuntu(the resolution is hardcoded at a bigger value, and my second old 17" LCD display flashes like hell)...

A nice feature I noticed is that yum allows installation of software from other arch'es (like x86 on x86-64), which APT forbids, and that yum is moving faster than I expected (it was damn slow in Centos on some of my servers). Still, zsh completion is unusable with yum, and queries in the Yum GUI tool could benefit from some speed-ups.

The only major thing that I'm missing currently is the VPN pptp client, which fails with "no valid secrets" even if my password is correct. It works just fine in Ubuntu.

Their "free software only" policy is not quite appreciated by pragmatic users like me, who just want the stuff work, despite the licenses are not that open, so I had to manually install EasyLife to get rid of this problem (It provides native 64bit Flashplayer and Java, Skype - whose sound is broken with Pulseaudio - and codecs among other nifty features).

That's about it for now, I'll update this as I find other issues/qualities as time goes by.

Cristi
