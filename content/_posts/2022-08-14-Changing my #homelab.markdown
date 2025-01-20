---
layout: post
title:  "Changing My Homelab"
summary: "Re-do part 3,098"
author: justinmjohnson
date: '2022-08-14 14:35:23 +0530'
category: Homelab
img: /assets/img/posts/2022-08-14/feat-1.png
keywords: Homelab
permalink: /blog/homelab-changeup/
usemathjax: true
imgdate: 2022-08-14
---

{%- comment -%} Please delete below and place your page content here {%- endcomment -%}

Haven’t posted in a while, and that’s because after getting my power bill, I had to rethink my #homelab and the direction I wanted to go. While running enterprise equipment is great to learn and understand how it all works, I already have a grasp of that technology and honestly, everything is more #cloud focused. With that, containerization, docker, Kubernetes (k8s, k3s, etc), and scaling (both up and out) are more practical means for businesses both public and private sector.

So what does that mean for my homelab? A makeover. I could very well just buy more enterprise gear and scale out in order to run Kubernetes and learn the technology (or just run it in “software” with multiple virtual machines under one hypervisor, but that’s no fun), but I’m still left with some rather power hungry machines, my R520 and R420, when compared to newer “prosumer” desktops I could get for about the same money…

## Introducing New Hardware
So what’s going to be in my new homelab? Well the Dell R520 was hosting TrueNAS with 2x 8tb HDD’s and 4x 6TB HDD’s, utilizing ZFS (1 Pool. 3 mirrored VDevs) and had 96 GB’s of DDR 3 ram and 2x E5-2430’s (12 cores, 24 threads). This was slightly overkill for something running nearly strictly NAS duties (and minor Docker Duties) and I needed something that was more scaled back.
### R520 Replacement (New NAS Box)
![Desktop Computer](/assets/img/posts/{{page.imgdate}}/2.png){:data-align="center"}
![Inside of new NAS Box (OMV)](/assets/img/posts/{{page.imgdate}}/3.png){:data-align="center"}

This computer was a DOA “For Parts Only” computer from eBay for $20 shipped. A Cooler Master Elite RC-520 Case with the following specs:

- ASUS M5A78L-M PLUS/USB3 Motherboard (Socket AM3)
- AMD FX-4300 Quad-Core Processor w/ CPU Fan (seller did not know if Processor was included)
- Rear 120MM Fan
- 8GB DDR-3 RAM (2x 4 GB Sticks)
- 500 Watt Power Supply
- No Hard Drives

The eBay seller could not get it to POST. I took a risk which paid off (I really wanted just the case, these older cases are harder to find with the multiple HDD bays). I changed the CMOS battery and reseated the RAM, booted immediately!

Compared to my R520 this thing sips power (at idle, I was measuring 154 Watts on average!!!)

![R520 Reading… It’s not doing anything right now other than sitting idle](/assets/img/posts/{{page.imgdate}}/4.png){:data-align="center"}

### R420 Replacement (Hypervisor(s))
![HP Elitedesk 800 G3](/assets/img/posts/{{page.imgdate}}/5.png){:data-align="center"}
![Inside with SSD](/assets/img/posts/{{page.imgdate}}/6.png){:data-align="center"}

The new hypervisor boxes (yes plural) are the HP Elitedesk 800 G3’s. These are outfitted with:

- Intel i5-7500 Quad Core (4c/4t)
- 8 GB DDR-4 RAM
- 256 GB SSD (2.5″ SSD)
- DVD Drive

I’ve owned these PC’s before and the neat thing is the Intel AMT onboard, which allows you to have remote management (IPMI – Think of Dell iDRAC) barring that you have a dummy plug OR a monitor connected to the machine.

I have three of these boxes, which I have plans on clustering with Proxmox. I also plan on upgrading the RAM initially with another 8GB’s (each) for a total of 16 GB of RAM, and taking a 6TB drive from my NAS and putting it into each of these devices and running either Ceph, Gluster, or MinIO (I have to do more research on this). Yes this will be robbing TrueNAS of drives, but TrueNAS will honestly be going by the wayside as I am downsizing the NAS, but not the Data. More on this in the next blog post. This cluster will allow for failover and live migration of virtual machines, but most importantly, I am going to set up k3s on these physical machines with shared storage.
