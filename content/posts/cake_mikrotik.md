+++
title = "Cake on mikrotik"
date = "2022-10-09T10:11:14+02:00"
author = ""
authorTwitter = "" #do not include @
cover = ""
tags = ["mikrotik", "network"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = false
hideComments = false
+++
CAKE (Common Applications Kept Enhanced) or [tc-cake](https://man7.org/linux/man-pages/man8/tc-cake.8.html) is a relatively simple way of setting up AQM (Active Queue Management) on a home network. What I find quite different about cake than other such protocols is that a clear goal of this is to make it relatively easy to setup for 'normal' people. The main reason to setup cake for me on my home network was to try reduce [bufferbloat](https://en.wikipedia.org/wiki/Bufferbloat). Which is essentially a fancy term for latency increasing when networks become congested due to packets being queued sequentially. AQM attempts to figure out which packets should be prioritised.

CAKE is typically used on the WAN interface of your network, and typically makes sense on networks where the WAN is much slower than your internal network (LAN) which unless you have symmetrical multi gig fiber to your house (lucky you!) is probably what you have.

## Do you have a problem?

A simple way to figure out if you have a problem is to use this [tool](https://www.waveform.com/tools/bufferbloat) from waveform. This tool essentially rates your network latency with a simple to understand score. It's not ideal but it's nice and easy. You should use a computer on a wired connection that is faster than your WAN port, I used gigabit ethernet for my gigabit DOCSIS3.1 connection. I got a B. Not bad, not great.

## Setup on Mikrotik

Mikrotik isn't usually known for super simple setups - but in this case it's actually really not so bad. I'm using it on an [RB5009UG+S+IN](https://mikrotik.com/product/rb5009ug_s_in). It should be very similar on all mikrotik versions since ROS7.x but I'm on 7.6rc1 here. There is some reasonably good [documentation](https://help.mikrotik.com/docs/display/ROS/Queues) on the mikrotik wiki.

What we will do here is setup CAKE and then apply it to the WAN interface. I'll show you howto do it via ssh but you can obviously do it with winbox, webapi etc...

What is important here is to set up the bandwidth, I'm lazy and just chose the autorate-ingress feature. This seems to do an ok job with relatively stable bandwidths, you can of course set up something manual but the bandwidth on my cable connection seems to be quite dependant on time ranging from ~600mbit/s to ~930mbit/s. Then the overhead compenstation is quite important and cake has really [good documentation](https://man7.org/linux/man-pages/man8/tc-cake.8.html#OVERHEAD_COMPENSATION_PARAMETERS) on what parameters to use based on the connection type you have. Mikrotik unfortunately don't support the keywords so you have to set it yourself, I'm going for 18 overhead, 64 mpu and noatm. I also decided to enable nat though I'm not sure I fully understand the difference.

```{.sh}
/queue/type add name=cake-for-my-wan kind=cake cake-autorate-ingress=yes cake-overhead=18 cake-nat=yes cake-flowmode=triple-isolate cake-ack-filter=filter cake-rtt=100 cake-atm=none cake-mpu=64
/queue/interface set ether8-wan queue=cake-for-my-wan

```

With all this, I got an easy A+ :). Good luck to you!
