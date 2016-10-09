---
author: mcristi
comments: true
date: 2016-07-13 23:52:57+00:00
layout: post
link: https://mcristi.wordpress.com/2016/07/14/autospotting-now-handles-complex-launch-configurations-when-replacing-your-ec2-instances-with-cheaper-spot-ones/
slug: autospotting-now-handles-complex-launch-configurations-when-replacing-your-ec2-instances-with-cheaper-spot-ones
title: AutoSpotting now handles complex launch configurations when replacing your
  EC2 instances with cheaper spot ones, and also got open-sourced.
wordpress_id: 825
categories:
- AutoScaling
- AWS
- EC2
- Spot
---

**Later Update: The code is now available on Github: [https://github.com/cristim/autospotting](https://github.com/cristim/autospotting)**

Today I finally reached a great milestone: for the first time I was able to get AutoSpotting provide spot instance replacements for AutoScaling groups of on-demand instances having full-blown real-life configurations with things like IAM roles, enhanced monitoring and attached IP addresses.

For those of you who are not familiar with it, have a look at this [presentation](http://talks.godoc.org/github.com/cristim/autospoting-cloudy-gophers-talk/autospoting.slide) as well as my previous [blog](https://mcristi.wordpress.com/2016/04/21/my-approach-at-making-aws-ec2-affordable-automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances/) [posts](https://mcristi.wordpress.com/2016/04/27/automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances-seeing-it-in-action/) where it's explained what it does and it is presented in more detail.

In addition, for those folks who are still running stuff on EC2-Classic environments, the latest build now also supports EC2-Classic security groups, which means that EC2-Classic works as well.

In order to test all this for real, I enabled it on an existing development environment running on EC2-Classic, which from the infrastructure perspective happens to be configured almost identically to those that are serving https://maps.here.com, and I'm happy to say that it worked like a charm:

![autospotting](https://mcristi.files.wordpress.com/2016/07/autospotting.png)

In the image above you can see a screenshot taken while replacing the group's instances.

Notice how in eu-west-1c we actually got a m1.medium instance which was chosen in order to spread to multiple instance types because at that time we used to have another m3.medium instance in that Availability Zone, since choosing the same instance type on too many machines may become risky.

Currently the algorithm prefers the cheapest instance type, but in order to avoid placing all the eggs in the same basket, when we have more than 20% of the total group's capacity of the same spot instance type within a single Availability Zone, the next cheapest instance type from that zone is chosen in order to reduce the chance of simultaneous failures of too many instances in case of sudden price fluctuations.

To make things even more interesting, during the replacement process one of the new spot instances failed to be fully configured and didn't become healthy when its grace period was over(we just happen to have an overkill setup process running at instance startup which sometimes fails to finish during the allotted grace time), so it was terminated by AutoScaling immediately after being added to the group. AutoScaling soon replaced the failed instance with another on-demand instance, later to be replaced by a new cheaper spot one. But eventually the group converged to a fully spot configuration.

Also because the group's scaling policy is currently based on CPU usage and has a quite low threshold, in the middle of all this replacement process a high CPU alarm fired due to the high load caused by the bootstrap of one of the new spot instances, so another new instance was launched by AutoScaling, only to be replaced by a new spot instance that was later teminated by a subsequent scale-in operation.

Eventually all this churn ended, and a group that would previously cost about $98 on a monthly basis, would now cost less than $17 assuming the price remains stable, which is more than 5.5 times cheaper on the long term.

So all in all it looks pretty good and reliable enough for dev environments (but I wouldn't immediately put it in production) and it allows for huge cost savings. Feel free to give it a try using these [instructions](https://mcristi.wordpress.com/2016/04/27/automatic-replacement-of-autoscaling-nodes-with-equivalent-spot-instances-seeing-it-in-action/) and let me know if you have any issues.

Before anyone asks, the software is not yet open sourced, but the review process is advancing fast and some important approvals are already there, so it's now a matter of just a few more weeks.

Many of the latest improvements were developed with a lot of help from [@nmeierpolys](https://twitter.com/nmeierpolys). His bug reports, suggestions and patience during multiple rounds of testing were priceless, and I am very thankful for all his contributions.

Known issues:



	
  * <del>It is currently broken for environments where the instances are set up depending on information set on their EC2 tags. This is due to the fact that currently the instance tags are set on the new instances very late, at the same time when the new instance is added to the AutoScaling group. So in case the user_data script depends on information derived from the instance tags, the information would very likely be missing at the time the instance runs the user_data script and the instance would fail to be configured. I am planning to set the EC2 tags much earlier, but your user_data script shouldn't rely they are there when the instance was started.</del>

	
  * The issue mentioned above was fixed as of July 17. The EC2 tags are now set as soon as the new spot instances are launched.


