---
title: "OPNsense IPv6 tunnel with Hurricane Electric TunnelBroker"
date: 2024-01-19T13:55:00-08:00
draft: false
tags: ["opnsense"]
---

## Background

My ISP, Ziply Fiber, still doesn't support IPv6. It's been talked about a lot on [reddit](https://reddit.com/r/ziplyfiber) over the past few years and folks have been nagging the nice folks at Ziply about it... but for reasons that I've basically determined are "they inherited a gigantic turd from Frontier Communications and further Verizon" are what the hold up is. Also that the demand for IPv6 is pretty low. (They will give it to you if you buy 10gig service... but as badly as I want that... I have no reason and no equipment with which to go above 1 gig).

So I wanted to give [Hurricane Electric TunnelBroker](https://tunnelbroker.net) a try again. Last time I tried, I think I was in college or high school?! And it wasn't very fast. But I saw somewhere on the ZiplyFiber subreddit that they're directly peered together at [SiX](https://www.seattleix.net/) so I figured I might have a better chance now than when I was on Time Warner Cable in Wisconsin and attempting to get peered to them at the Chicago exchange.

## Notes

1. Go sign up at https://tunnelbroker.net/ and do the minimum level of certification required to be *HE.NET IPv6 Certified*. To be honest, I don't remember what that took, but I don't think it took long.
1. When it's done, from the dashboard, do *Create Regular Tunnel*.
1. Enter the WAN IPv4 for the OPNsense gateway and pick the closest endpoint, Seattle in my case. Then create tunnel.
1. Very important: Under **Routed IPv6 Prefixes**, look at *Routed /48*. It should have a link that says to issue one. Do that.
    - **Why?**
       -  If you use the *Routed /64* they give you, all Google services will be suspicious of you. Turns out Google just blacklists the entire range that those are a part of. In my case, it meant Google kept making me pass captchas for darn near everything and that the Google homepage kept redirecting me to the `com.hk` variant no matter what.
       - If you're anything like me, you have multiple subnets to route anyway... and a `/64` in IPv6 parlance is only one subnet worth... even though it's absolutely massive, it's the smallest unit of network division.
    - **CAREFUL!**
       - You can only issue one of these per 24 hours. If for some reason you deleted it or messed with it... you have to wait 24 hours before you can make a new one. It'll give you some sort of limit error. Guess how I figured that out. 
1. Now you can mostly follow the directions at https://docs.opnsense.org/manual/how-tos/ipv6_tunnelbroker.html.
    - **Step 3 - Basic Firewall Rules** - This was completely unnecessary for me as outbound connections are allowed by default and inbound are denied by default. I didn't intend to pass anything in on the IPv6 side right now so I skipped it.
    - **Step 4 - Configure LAN interface** - OK, so you just pick one. You were given a */48* which contains 3 segments of the IPv6 address (`seg1:seg2:seg3::/48` is what you should see in the panel). So just make something up for the fourth segment, maybe vaguely matching your v4 numbering, and go with it. I did `seg1:seg2:seg3:1::/64` for my `192.168.1.1/24` network figuring I would do `seg1:seg2:seg3:40::/64` for my `192.168.40.1/24` network and so on. They blacked this out completely in the directions and left you to figure it out.
    - **Step 5 - Configure DHCPv6 SLAAC** - This doesn't work for me. No matter what I would try for the range off of the `/64` I created from the `/48`, it wouldn't start. I went to `System > Log Files > General` and it said `The specified range lies outside of the current subnet.` Searching the internet for that gets you [this forum post](https://forum.netgate.com/topic/175434/dhcpv6-configuration-the-specified-range-lies-outside-of-the-current-subnet) among others for pfsense/opnsense that say this is frustrating but have no results. So don't set up DHCPv6. Instead, go to **Services > Router Advertisements** and set it to **Unmanaged**. All the clients will make something up for themselves in the range and it's just fine. Ignore the **Services > DHCPv6 > LAN** entirely and leave it off.

Ta-da. It just works. I went to [WhatIsMyIP](https://whatismyip.com) from my phone and laptop and desktop and they just have a public IPv6 now. Speed test from my phone didn't seem to show any slow down at all. If you use [Tailscale](https://tailscale.com) from outside the network (like on a phone) and set the gateway as the exit node, you predictably see its public IPv6.