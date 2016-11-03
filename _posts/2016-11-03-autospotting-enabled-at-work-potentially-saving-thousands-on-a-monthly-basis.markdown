---
author: cristim
comments: true
layout: post
categories:
- AWS
- EC2
- Spot
---

It took quite a while, but today I hit a huge milestone, I finally managed to
deploy my [AutoSpotting](https://github.com/cristim/autospotting) project at
work for the first time on a few non-experimental (but still non-production)
environments which would hopefully save my company a few thousand dollars on a
monthly basis with little risks.

It took me just a couple of minutes to launch the CloudFormation stack in two of
my AWS accounts so that I could go ahead and enable it on a few of environments.

One of the earliest AutoSpotting
[issues](https://github.com/cristim/autospotting/issues/4) on GitHub, open for a
really long time was asking for it to be tested on EC2 Classic, so I thought it
would be nice to first deploy it on such a configuration, and later test it on
bigger VPC environments.

# EC2 Classic

To make it more challenging, I picked a quite important environment, the staging
environment of the [HERE WeGo website](https://wego.here.com), the main
application I am working on on a daily basis. This would be normally the easiest
to handle by the tool because it's running in a relatively quiet region on the
spot market, with quite stable prices over the last few months. Since as a
staging environment it takes quite low traffic, it was set up to run just a
couple of m3.medium instances. Since AutoSpotting is designed to be failing in a
harmless manner whenever possible, I thought this would not be a big problem,
but clearly I was wrong.

## Bug hunting

After enabling AutoSpotting, I noticed that nothing happened and no spot
instances were launched. I had a look at the logs and I noticed instance
launches would throw an error due to the handling of Security Groups on EC2
Classic. On VPC the SGs are set on a network interface object, which makes no
sense on Classic, which has a different mechanism. Since Classic accounts are a
rarity nowadays, this is not a very tested configuration and I was more or less
expecting this kind of issues, especially since I touched the same code a while
back when I had fixed the same for VPC.

I soon [fixed](https://github.com/cristim/autospotting/commit/1e189a2256edcfe5800059f6a039d9f4efcdd47a)
it for EC2 Classic as well, with just a few lines of code, I ran it locally and
then after pushing I updated the AutoSpotting stack. The fixed version soon
replaced both instances with the cheapest available compatible instance, which
in this case happens to be m1.medium.


Mission accomplished! AutoSpotting is now confirmed to be working on EC2
Classic.

## Results

If the spot prices in this region keep being like in the last three months, I
expect this simple environment to cost about $12 monthly instead of $111, saving
almost $100 monthly. We have literally dozens more like this, so the savings
will add up quickly to significant amounts.

# VPC

The other environments where I enabled it are a number of significantly bigger
development environments of a couple of other backend applications I've been
working on for a while now. They are all used as ECS hosts and running in a VPC
in US-East-1:

- 2 identical environments of 10 instances each, both with m3.xlarge instances
- 1 newer environment consisting of a single m3.xlarge instance
- 3 identical environments of 3 instances each, all with t2.medium instances

## Second bug

Even if I had intensively tested AutoSpotting on VPC environments before and I
didn't expected any surprises, I again noticed an issue that resulted in
instances failing to launch(this time simply because no compatible instances
could be found), reminding me how important is to test it in the wild.

After looking into it, I noticed the root cause was an unhandled edge case
related to our quite 'creative' instance store volume mapping configuration.

Our Launch Configuration's storage configuraiton is specifying four instance
store volumes, just in case we ever change the instance type for a bigger one
that has more instance store available. When launching a m3.xlarge instance
which only provides two instance store volumes, the other two ephemeral volumes
set in the launch configuration would simply be ignored and the instance would
only expose the available volumes.

AutoSpotting's quite sophisticated instance compatibility check is comparing the
new spot instance type's total number of available instance store volumes
against the number of ephemeral volumes specified in the launch configuration,
silently assuming that their number would never exceed the number of instance
store volumes available for the current instance type.

The original idea behind this check was allowing the algrithm to choose any
other instance type that sports the number of volumes actually used by the
launch configuration, even if the original instance has some unmapped ephemeral
volumes, since I never expected the launch configuration to have more ephemeral
volumes than available on the instances.

I immediately changed the algorithm to compare the new spot instance's number of
ephemeral volumes with the minimum between the number of volumes specified in
the launch configuration and the number of volumes available on the original
instance.

Once this was fixed, instances started being replaced and I soon got all these
environments backed by spot instances.

## VPC Results

The t2.medium instances in all those three groups were soon replaced with
m3.large, which are a number of times more powerful, cost about half the price
and their price over the last three months was relatively stable, at least in
three of the four available Availability Zones. Over those t2.medium
environments I estimate about $175 monthly savings if the prices remain stable
at the current levels.

On the bigger m3.xlarge environments it is a bit harder to evaluate and predict
future savings, because they are big enough that the algorithm tries to spread
capacity over at least four different spot cost configurations(I often call
these 'cost zones'). Three of them are set for the original m3.xlarge instance
type, one in each of the three availability zones that we use, but the options
we have for the fourth are quite narrow due to our storage configuration, which
pretty much only allows for m3.2xlarge or bigger instances.

Since the Virginia region is much more volatile on the spot market for those
larger instance types, for the last few hours since I enabled AutoSpotting
there, I saw a number of instance terminations and replacements. We briefly even
got a massive i2.2xlarge which is a lot bigger than what we started with, and
their price tends to be very close to the original m3.xlarge ones. For a while
we even had some on-demand instances left because as per the current algorithm
we had no acceptable options available on the spot market on this fourth cost
zone.

But for the bulk of the m3.xlarge spot instances the price seems to be quite
stable and historically there were relatively few terminations, so assuming this
holds true in the future, I estimate about $2700 savings on a monthly basis
for them, which is already significant.

Going forward, I would like to have a look at our storage configuration, since
we may not need to have any instance store volumes for this environment. This
change would widen the range of options for the fourth cost zone, likely
providing a cheaper and more stable alternative instance type which may save us
a few more hundred dollars.

# Conclusions

All in all, enabling AutoSpotting on these dev environments alone would allow us
to save about $3000 on a monthly basis(again, assuming the market prices remain
at current levels), which is not bad at all for just a couple of hours spent
doing bugfixes. This is a tiny fraction of our massive infrastructure of
literally thousands of instances. I will definitely report more results once I
enable it on more of our environments.
