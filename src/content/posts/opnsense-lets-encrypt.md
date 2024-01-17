---
title: "OPNsense with Let's Encrypt and IONOS"
date: 2024-01-17T15:31:00-08:00
draft: false
tags: ["networking"]
---

I've tried a few different ways of getting SSL certificates onto OPNsense including using the one provided by IONOS as a part of my domain. Which is mostly fine. Until the annual renewal comes up.

It's also a wildcard certificate which worked okay for all my other services.

Except I decided to add another level of hierarchy to my internal domains so each router/location would have its own `device.location.domain.com` instead of every device being `device.domain.com`. The wildcard certificate no longer worked when I did that.

However, when I did Nginx Proxy Manager the other day, it looked like IONOS is finally supported for DNS-01 challenges for automatic updates.

Turns out, the one in OPNsense is too.

I was going to build a guide, but I found a [pretty good guide](https://homenetworkguy.com/how-to/replace-opnsense-web-ui-self-signed-certificate-with-lets-encrypt/) that did exactly what I want... just substituting his Cloudflare choice for my IONOS one. And it was that easy.

Just for IONOS, you have to go to [their developer site](https://developer.hosting.ionos.com/keys) and make a public prefix and secret pair. 
