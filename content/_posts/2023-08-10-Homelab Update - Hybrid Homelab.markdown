---
layout: post
title:  "Homelab Update - Hybrid Homelab"
summary: "Unless you have been living under a rock, you know that the Cloud is one of the biggest things in IT, and if you aren't already learning, working, or developing yourself in a cloud-based environment, you are not only missing out, you are doing yourself a disservice."
author: justinmjohnson
date: '2023-08-10 14:35:23 +0530'
category: homelab
img: /assets/img/posts/2023-08-10/1.png
keywords: HomelabUpdate, Hybrid, Cloud
permalink: /blog/Homelab-Update-Hybrid-Homelab/
usemathjax: true
imgdate: 2023-08-10
---


### Preface:
Unless you have been living under a rock, you know that the Cloud is one of the biggest things in IT, and if you aren't already learning, working, or developing yourself in a cloud-based environment, you are not only missing out, you are doing yourself a disservice. (I can hear some of you guys now: 'Justin, the cloud is really just someone elses computers' .... Yes, yes I know). The associated cost with running a homelab in the cloud, especially to any sort of extent, can be a deterring factor for most, however Oracle provides (at the time of this writing) free cloud instances, as long as you remain within the free tier. Best of all, you can connect this instance to your home environment and pass services back and forth, creating a hybrid environment - where you would have on-premise devices connected to the cloud (not to be confused with Azure Hybrid AD, or a Hybrid Cloud model which would combine Public and Private cloud). 

Oracles [https://www.oracle.com/cloud/free/](free tier) provides two 1 CPU/1GB AMD servers and up to 4 instances of ARM Ampere A1 compute devices (with upwards of 24 GB of ram!!!!). Currently I have 4 instances running and have been running this configuration for over a year, and recently I have it connected to my homelab.


![Oracle Cloud Logo](/assets/img/posts/{{page.imgdate}}/1.png){:data-align="center"}

#### Oracle connected to ESXI
So inside of the homelab I currently have three resource pools running. One of these pools is a separated homelab (not listed), however one pool is a test bed for currently testing new tools (Labeled DMZ-TestBed) and one is a mirror of my public cloud interface that I am currently setting up (CloudMirror):
![Two resource pools](/assets/img/posts/{{page.imgdate}}/2.png){:data-align="center"}

And within Oracle I have four (4) cloud instances running - Demo and Demo-Docker (These connect to my DMZTestBed resources) and Proxy and DockerCloud (These connect to my CloudMirror Resource Pool):
![Four Instances](/assets/img/posts/{{page.imgdate}}/3.png){:data-align="center"}

And below is a rough diagram of how this all works.
![Diagram](/assets/img/posts/{{page.imgdate}}/4.png){:data-align="center"}
(I obviously do not draw diagrams for a living else I would be broke).

Now how do I connect these together? Starting with the CloudMirror -> Proxy and DockerCloud I am using Tailscale with a MeshVPN solution. This allows me to not open ports into my home and makes it easy to use a third party "Control node" to control access from the outside. I create a site to site vpn which can communicate between my cloud environment and my home lab environment so that each device sees each other on a local network (i.e. I can ping the PRIVATE IP of the other device on the other side of the tunnel). This makes it REALLY nice for future growth of the lab. The virtual machine currently running in ESXI is a near exact replica of the Oracle Cloud instance, aside of the fact that Oracle is running ARM and my VM is running on x86_64, otherwise the same size SSD, RAM, etc. This is to keep the environments as close as possible for the docker environment I have installed on each.

I'm running portainer on each environment and have installed the portainer agent on each environment, therefore I can log into Portainer on one side (or the other, if the cloud port was open or I was logged in via the VPN) and control the other node. This also makes copying the docker containers easier. From there I can rsync (copy) the containers volumes/data to the other node with ease so that any updates I make in "pre-prod" (home) can be pushed to prod (cloud). Neat.

#### DMZTestCloud
Here I am using this environment mostly for work related tasks (Splunk Dashboards, STIG testing, future products, changes, etc). Right now (on some off time) a product being tested is [https://goteleport.com/](Teleport), which allows remote access through CLI, Remote Desktop (think of a Virtual Desktop Infrastructure) and remote applications. Teleport also allows connections without the use of a VPN making it really nice to work with as well. 

This environment is also separated by the pfSense Firewall from the rest of my network (My Network -> DMZ vLAN -> pfSense Firewall (Heavily Firewalled to no end) -> Test Environment). This is due to wanting to be able to allow other users access as needs arise (other nerdy friends), and due to the fact we may end up breaking something/introduce something bad that breaks everything - I really don't want to take down anything else on my network. 