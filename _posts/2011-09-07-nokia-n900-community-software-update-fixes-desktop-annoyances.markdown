---
author: mcristi
comments: true
date: 2011-09-07 20:45:38+00:00
layout: post
link: https://mcristi.wordpress.com/2011/09/07/nokia-n900-community-software-update-fixes-desktop-annoyances/
slug: nokia-n900-community-software-update-fixes-desktop-annoyances
title: Nokia N900 community software update fixes desktop annoyances
wordpress_id: 77
categories:
- N900
---

I love my N900 ever since I bought it, it's a great device for a nerd like me. Still, as nothing is perfect in this world, I has some things that I don't like that much about it.

Today I will address two of them, more exactly the fact that the items on the desktop could be placed everywhere, and  the other is the fact that the desktop is forced to landscape mode, when there were many applications that also work in portrait mode.

I am glad to report that today I finally got both of these annoyances fixed on my beloved device, and here's how I did it.

This morning I applied the latest [Community SSU update](http://wiki.maemo.org/Community_SSU), which I soon found out that it introduced the support for portrait mode on the desktop. This is very nice stuff, and very easy to use. After applying the update, just rotate the device to portrait mode (when the keyboard is hidden) and you will see all of the content switch to portrait mode. This is not so nice at first, because everything is messed up, but you only need to move the items around and after you switch back and forth between portrait and landscape modes, they will remember where you put them in both orientations. Problem solved!

Besides this issue, as I said, I never liked the fact that moving items was not constrained by anything on the N900 desktop. This looked especially bad after moving all my items to more or less acceptable positions when in portrait mode so that they won't overlap. After this process, the desktop looked like hell having all those icons unaligned. I shortly got this problem fixed, after applying a suggestion I got from one of the people in the #maemo-ssu IRC channel. The solution was to edit /usr/share/hildon-desktop/transitions.ini and set the following options:

    
    snap_grid_size = 20
    snap_to_grid_while_move = 20


There's currently no UI for these settings from what I know so far, but I would really appreciate if these were included in the cssufeatures application if someone cares enough to do it.

Feel free to use any values  you might see fit, but in my case it worked just fine with 20. After rebooting the device, moving the items on the desktop would align them into a grid, so my desktops look much better now, as you can see in the screenshots below.

[![](http://mcristi.files.wordpress.com/2011/09/screenshot-20110907-231355.png?w=300)](http://mcristi.files.wordpress.com/2011/09/screenshot-20110907-231355.png)

[![](http://mcristi.files.wordpress.com/2011/09/screenshot-20110907-232130.png?w=180)](http://mcristi.files.wordpress.com/2011/09/screenshot-20110907-232130.png)

The vertical screenshot could only be taken while the desktop was in edit mode, because otherwise the screen would switch to landscape when the keyboard is visible, and I needed keyboard in order to get the screenshot. I know it can be done from the command line, but I was just too lazy.

I hope this is useful to someone. Feel free to post comments to this post containing additional fixes to annoyances you might encounter on this device.

Thank you for reading this and thanks to all the CSSU developers who made this possible.

Cristi

Update: It seems there was yet another minor CSSU release a few hours after the one I was talking about.

Update2: I now discovered that the cssufeatures application is incompatible with the manual changes I did to transitions.ini.


