---
title: "Move from Azure CDN to Cloudflare Pages"
date: 2025-01-02T21:25:00-08:00
draft: false
tags: ["site"]
---

# Migration from Azure CDN to Cloudflare Pages

## Background
I was informed over and over the last few weeks about the [imminent closure](https://www.theregister.com/2025/01/02/microsoft_endpoint_change/) of [Azure CDN by Edgio](https://learn.microsoft.com/en-us/azure/cdn/edgio-retirement-faq).

I mostly didn't care and was going to let them try to migrate for me. But when I published the earlier page today about my [Ikea Tradfri Button Fun]({{ ref "tradfri-shortcut.md" }}), I noticed that the good ol' [key expired]({{ ref "renew-azure-key.md" }}).

So. Since this site is barebones and since I spent 2024 trying a handful of services from Cloudflare because Google Domains [sold out](https://www.theregister.com/2023/06/18/google_domains_shutting_down/), I decided to try setting up pages on Cloudflare too.

Other reasons I had for moving to Cloudflare
- They don't seem to charge markup on domains and did support .dev, .com, and .net which are the ones I own one way or another. (They don't support all TLDs)
- They have a ton of services included in the free tier
- Basically every Let's Encrypt ACME DNS challenge client I can find built into any other software I'm using for whatever reason has Cloudflare's API built in natively without headache whereas the Azure DNS one is either not there, requires me to find and load a plugin, or is occasionally a preinstalled plugin and more of a headache.

## Process
1. Go to [Cloudflare Pages](https://pages.cloudflare.com/) and sign in with a Cloudflare account in the top right.
2. Hit the **CREATE** button in the top right.
3. Switch tabs from *Workers* to *Pages* at the top.
4. Click **Connect to Git** button and do the OAuth jig with my GitHub account
5. When it comes back, choose the *site* repository where I keep my site and press **Begin Setup** in the bottom right.
6. Use default project name and default branch (`main` which is what I use). Set *Framework preset* to *Hugo*. Change my *Build output directory (advanced)* to `/src/` because I keep my Hugo files down one directory in that repo and hit **Save and Deploy**
7. It just worked. They pulled and built and deployed it completely to https://site-6ut.pages.dev. Well it mostly worked. It took like 3-5 minutes for that DNS name to actually resolve. But I did nothing.
8. Clicked their suggestion of more things to do that said **Custom Domain** and picked the `niksa.dev` one I had already put there. It suggested to delete all my various `A` records and make a new `CNAME` from `@` (the root) to `site-6ut.pages.dev` and so I hit OK.
9. It's done I swear.

## Conclusion
I put less than 3 minutes of work clicking a few buttons of a wizard into migrating from my GitHub Actions/Azure Blob Storage/Azure CDN stack for running my statically generated Hugo website to Cloudflare Pages. The DNS propagation and the OAuth to GitHub were the slowest parts of the whole thing. It probably would have taken longer had I not already moved my domain and DNS into Cloudflare earlier this year, but that wasn't hard either. And now I can just go delete my Azure CDN by Edgio instead of figuring out how to migrate it or deal with the expiring key vault. There's a reason Cloudflare is kind of a winner right now I suppose.
