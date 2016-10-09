---
author: mcristi
comments: true
date: 2016-04-21 01:07:27+00:00
layout: post
link: https://mcristi.wordpress.com/2016/04/21/my-approach-at-making-aws-ec2-affordable-automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances/
slug: my-approach-at-making-aws-ec2-affordable-automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances
title: 'My approach at making AWS EC2 ~80% cheaper: Automatic replacement of Autoscaling
  nodes with equivalent spot instances'
wordpress_id: 94
categories:
- AutoScaling
- AWS
- EC2
- Spot
---

# Update


The tool was given the name AutoSpotting, was open sourced and is now available on [GitHub](https://github.com/cristim/autospotting).


# Getting started


Last year, during one of the sessions of the [Berlin AWS meetup](http://www.meetup.com/AWS-Berlin/) where I am often present, during the networking that happened after the event [@freenerd](https://twitter.com/freenerd) from Mapbox mentioned something about the [spot market](https://aws.amazon.com/ec2/spot/), saying how much cheaper it is for them to run instances there, but also the fact that for their use case it sometimes happened that the instances were terminated in the middle of their batch processing job that prepares the map for the entire world.

A few weeks later, at another session of the AWS meetup, I participated in a similar discussion where someone mentioned the possibility to have instances attached to an on-demand AutoScaling group, which was a feature just released by AWS at that time. I don't remember if spot was mentioned in the same discussion, or if it was all in my mind, but somehow these concepts got connected and I thought this is a nice problem to hack on.

I was thinking about the problem for a while, and after a couple of weeks I came up with an algorithm based on the instance attach/detach mechanism supported by AutoScaling. I tested it manually and I quickly confirmed that AutoScaling happily allows attaching spot instances and detaching on-demand ones in order to keep the capacity constant, but that it often tries to rebalance the availability zones, so in order for it not to interfere with the automation, the trick is to try to keep the group more or less balanced across availability zones, so that AutoScaling won't try to rebalance it.

I soon started coding a prototype in my spare time, which is actually my first non-trivial program written in a while, and to make it even more interesting, I chose to write it in golang.


# Slow progress


After a few weeks of coding, in which I rewrote it at least twice(and even now I'm still nowhere near being happy with how it looks), I realized it's quite a bit harder and more complex than I initially thought. Other things happened and I kind of lost interest, I stopped working on it and it all got stuck.

A few months later at the re:invent conference I attended some talks where I met some other folks interested by this problem and I saw other approaches of attacking the problem, with multiple AutoScaling groups, and that was also when I first got in touch with someone from [spotinst.com](http://spotinst.com) who was trying to promote their solution and was sharing business cards.

After re:invent I became a bit more active for a while, I also tried to get some collaborators but failed at it, so I kept working on it in my spare time every now and then and I got closer to get it work. Then I recently had a long vacation, and immediately after I returned I attended the Berlin AWS Summit, where I met the SpotInst folks once again, and it seems they now have a full fledged solution for the problem, based on pretty much a reimplementation of AutoScaling, using machine learning and with a beautiful UI and they are really successful with it. Funnily enough, they even contacted me to sell that solution to my company and we are seriously evaluating it :-)


# Breakthrough


After the Berlin AWS Summit, having my batteries charged, I resumed my work and after a few coding nights I managed to make my prototype work. It took much longer than expected, but at least I got there, yay! :-)


# What I have so far





	
  * An [easy to install](https://mcristi.wordpress.com/2016/04/27/automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances-seeing-it-in-action/) CloudFormation [template](https://s3.amazonaws.com/cloudprowess/dv/template.json) that creates an SNS topic, a Lambda function written in golang(with a small JS wrapper that downloads and run it), subscribed to the topic and a few IAM settings to make it all work

	
  * A golang binary, for now closed source, but I'm going to open it up once I get it in a good enough shape so that I'm not ashamed of it and after I get all the approvals from my corporate overlords, who according to my employment contract need to approve the publishing of such non-trivial code





# How does it work


The lambda function is executed by a custom CloudFormation resource when creating the CloudFormation stack from the template, and it subscribes to both your topic and a topic that I run, which fires it every 30 minutes, using a scheduled event.

When my scheduled function runs the lambda function, it will concurrently inspect the AutoScaling groups from all the AWS regions and it will ignore all those that are not tagged with the EC2 tags it expects.

The AutoScaling groups marked with the expected tag will be processed concurrently, on each of them gradually replacing the on-demand instances with compatible spot instances, one at a time. Each run will either launch a single spot instance or attach a launched spot instance to the AutoScaling group, after detaching an on-demand one it is meant to replace. The spot instance is not attached while its uptime is less than the Autoscaling group's grace period.

The spot instance bid price matches the price of the on-demand instance it is meant to replace. If your spot request is outbid, AutoScaling will handle it as a regular instance failure, and will immediately replace it with an on-demand instance. That instance will later be replaced by the cheapest available compatible spot instance, likely of a different type and with a different spot price.

In practice the group should converge to the most stable instance pricing, no the long term saving about 80% from the normal on-demand EC2 price.


# How to use it/Getting started


All you need to do is set an EC2 tag on the AutoScaling group where you want to test it. Any other AutoScaling groups will be ignored.

The tag should have the following attributes:

Key: "spot-enabled"

Value: "true"

See my next blog post for a [full installation and runtime walkthrough](https://mcristi.wordpress.com/2016/04/27/automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances-seeing-it-in-action/) where you can see very detailed instrunctions on how to get started.


# Feedback is more than welcome


If you find any bugs or you would like to suggest any improvements, please comment below.


# **Warning**


This is experimental, summarily tested and likely full of bugs, so you should not run it on production, but it should be safe enough for evaluation purposes.

Anyway, use it at your own risk, and don't hold me responsible for any misuse, bugs or damage this may cause you.


# **Later Updates**





	
  * 27 Apr 2016

	
    * if you want to see it in action, please also check out my [next blog post](https://mcristi.wordpress.com/2016/04/27/automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances-seeing-it-in-action/) which walks you in detail through the installation and setup process, also explaining the currently known issues and their workarounds as of the time of the writing of this post.




	
  * 5 May 2016

	
    * bug fixes since the previous update

	
      * no longer spinning the AutoScaling group when running at minimum capacity

	
      * increased runtime frequency to once every 5min to make it converge faster




	
    * currently known issues

	
      * when first enabling it on a group with multiple on-demand nodes, it sometimes may launch extra spot instances that do not get added to the AutoScaling group(up to as much as the initial size of the group). workaround: terminate them manually from the AWS console. They will not be re-launched

	
      * spinning condition when the AutoScaling group is set to a fixed size (Min=Max). Workaround: set Max to Min+1 and disable any scaling actions you may have configured, in order to keep the group at the minimum capacity




	
    * things currently being worked on

	
      * fixes for the known issues mentioned above

	
      * major under-the-hood code refactoring in preparation of open sourcing

	
      * choosing instance types that are unlikely to be terminated in the near future, based on historical stats data sourced from the [Spot Bid Advisor](https://aws.amazon.com/ec2/spot/bid-advisor/)

	
      * mark spot instances as protected from termination by AutoScaling

	
      * if you have any other suggestions please write a comment below.







	
  * 10 July 2016

	
    * new features

	
      * improved algorithm for picking the new spot instance type

	
        * always launch a new spot instance  from the same zone of an existing on-demand instance. Previously the zone was the one where we had the less instances, which often may have been one where we had no running instances and no way . This was causing the bugs about spinning and launch of additional spot instances that were not added to the group.

	
        * allow multiple spot instances of a given type for each availability zone, as long as their total number is less than 20% of the total capacity from the group. For example a group of 15 instances using 3 availability zones will allow for 3 identical instances per availability zone, but the fourth instance from a zone will be of a different instance type.




	
      * Lambda wrapper updates

	
        * rewrote the Lambda wrapper in Python, which makes it more maintainable, since I'm much better at Python than at JavaScript

	
        * Implement versioning for the binary blob, by downloading the latest version only if not already there, based on the content SHA hash




	
      * CloudFormation cleanup

	
        * remove the SNS topic that was never used




	
      * support the new Mumbai region(still needs testing)

	
      * internal code refactoring

	
        * lots of cleanups that make it more maintainable

	
        * improved logging







	
    * bug fixes since the previous update

	
      * it should no longer start additional spot instances, the final capacity should match the original on-demand capacity, unless there were any AutoScaling actions.

	
      * fixed spinning condition with fixed-size AutoScaling groups by temporarily increasing the group during the replacement process

	
      * fix choosing of the cheapest compatible/redundant enough spot instance type, previously any cheaper instance type may have been chosen, not necessarily the cheapest.

	
      * lots of other small bugfixes for various edge cases




	
    * things currently being worked on

	
      * open sourcing process was started, and I already got some of the required approvals

	
      * figuring out how to implement automated testing




	
    * backlog

	
      * choosing instance types that are unlikely to be terminated in the near future, based on historical stats data sourced from the [Spot Bid Advisor](https://aws.amazon.com/ec2/spot/bid-advisor/)

	
      * mark spot instances as protected from termination by AutoScaling







	
  * 13 July 2016

	
    * Mostly bugfixes, many thanks to [@nmeierpolys](https://twitter.com/nmeierpolys) for some very valuable bug reports, fast feedback and a lot of patience while testing it.

	
      * improved conversion of on-demand launch configuration fields into spot launch configuration equivalents. In addition to user_data, SSH keys, EBS volumes and many other mostly trivial to convert fields that were previously handled, the following more complex fields should now also be better handled, which make it work on much more real-life environments:

	
        * EC2 Classic security groups

	
        * detailed instance monitoring

	
        * associating public IP addresses




	
      * It was successfully tested on complex EC2-classic and VPC setups where many of these fields were being used.

	
      * Compatibility notice: In the likely event that you are using IAM roles on your instances, you need to update to the latest version of the CloudFormation template, since the launch of such spot instances would otherwise fail due to missing IAM permissions required to run instances set up with IAM roles. Again thanks to [@nmeierpolys](https://twitter.com/nmeierpolys) for finding out this issue and proposing the fix.







	
  * 18 July 2016

	
    * Improve handling of storage volumes.

	
      * Bugfix: Fix panic while copying EBS storage configurations.

	
      * New feature: Implement compatibility check for storage volumes based on the number of attached ephemeral disk volumes present in the launch configuration. For example an instance which has a launch configuration that attached it a couple of the ephemeral SSD drives of a certain size would only be replaced by instance types which provide SSD devices at least as many and of at least the same size each, in order not to violate the storage expectations from the new instances.








