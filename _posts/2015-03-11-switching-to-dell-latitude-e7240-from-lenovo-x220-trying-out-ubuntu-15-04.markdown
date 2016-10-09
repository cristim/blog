---
author: mcristi
comments: true
date: 2015-03-11 21:37:26+00:00
layout: post
link: https://mcristi.wordpress.com/2015/03/12/switching-to-dell-latitude-e7240-from-lenovo-x220-trying-out-ubuntu-15-04/
slug: switching-to-dell-latitude-e7240-from-lenovo-x220-trying-out-ubuntu-15-04
title: Switching to Dell Latitude e7240 from Lenovo x220, trying out Ubuntu 15.04
wordpress_id: 89
categories:
- dell
- driver issues
- evolution
- gnome3
- hardware
- lenovo
- linux
- ubuntu
- unity
tags:
- Dell
- docking station
- Lenovo
- lock screen
- Ubuntu
---

I've been using a Lenovo x220 for the last few years as work laptop and I was pretty satisfied with it, it was a nice piece of hardware.

At some point 2 years ago I had a quite bad bike accident(out of which I was lucky to escape with just an elbow fracture and a few bruises, it could have been much worse...) and the laptop had to absorb a lot of the shock since I basically fell on the back while it was in my backpack. One of the corners was a bit bent and it got some cracks but it was alive and kicking, until last week I finally decided that I would like to have an upgrade.

I've had a few Ubuntu issues lately(unity's lock screen would often fail to allow me to enter a password, random error messages about program crashes, repeated password prompts, random unity panel disappearances) which I blamed on my aged installation, but the last straw was that I got out of disk space on my btrfs partition and the system got unusually slow. I soon saw kernel oops messages in dmesg and soon after MCE errors as well, which to my knowledge is a sign that the hardware is dying.

So I just went to our awesome IT support guys(yeah, I'm not doing that at HERE, we have a dedicated team for it) and after a few minutes of chatter in which I explained what happened, they quickly gave me the latest from our offering in the similar range, a Dell Latitude e7240. I almost took a Lenovo X1 Carbon, but I ended up choosing the Dell because it has support for a real docking station, unlike the Carbon, which is a must if you have a dual-screen setup like I do.

In general, the hardware looks much more polished, the screen is really much better and I could feel it's slightly faster than the X220. The only things I miss, and quite badly, are the classic Lenovo keyboard and the trackpoint, especially since the Dell trackpad sucks so badly.

I didn't feel like reinstalling the OS from scratch, especially since I have a quite exotic setup (btrfs on LVM, on top of a full-disk LUKS volume) which took me a while to figure out manually a while back, and restoring from a backup would be slightly slower, so I quickly copied my entire disk on it using dd and netcat, which only took about 45min to complete.

I immediately connected it to my screens using the docking station and tried to configure it so I can resume my actual work, only to notice that both the external monitors are showing the same content and there was no way to separate them.

I did some research and ended up on some forums that claimed this is a known driver bug on my graphics chip on kernels older than 3.17(and the darn Ubuntu 14.10 comes with 3.16). After using the X220 for 3 years with no major driver issues, I really had better expectations from Intel drivers, especially for a year old laptop running the latest available version of Ubuntu. The proposed solution was to update the kernel to at least 3.17, which I did immediately, only to notice that the new kernel fails to even boot, getting stuck while asking me the passphrase to decrypt my LUKS volume.

Since I pretty much had no choice, I then reverted to the previous kernel and decided to try to update Ubuntu to the next development release, which will be launched next month as 15.05, which already comes with the version 3.19 of the Linux kernel and should have the display problem solved.

I then had to free up some space, making room for the upgrade and let it do its thing for a few hours.

Once it was ready, I connected the screens, and it all worked like a charm.

I then thought to give Gnome3 another chance after a few weeks since I tried it last, hoping it would improve, but I was quickly disappointed by its brainless behavior on a triple-screen setup, where having a fixed primary monitor set to the right-most laptop screen really makes no sense(I personally think the primary should follow the mouse, just like in Unity).

I might give KDE another try at some point and I will give my impressions about it, but I will stick with Unity for now, especially since I really love the way they reuse the topbars as menubars on non-maximized windows, and that in generally it feels more polished than in 14.10.

As a bonus, I was pleasantly surprised that Evolution now has a smart push-notification-like update mechanism when used with Exchange, that makes it much more resource efficient than before at checking for new emails.

I'm using it for a few days now and things seem decent, actually surprisingly stable for a development version, but I still see the unity issues with the missing password field in the lock screen and the panel still disappears from time to time, so the issues are still there and not fixed yet.

I'll do a bit more research and hopefully I'll get to the bottom of them soon and report the findings in another post.
