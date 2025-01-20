---
layout: post
title:  "Homelab Update - OPNSense Splunk and Syslog"
summary: "As part of the Homelab revamp, I decided it best to completely blow away all of the VM's and start fresh. "
author: justinmjohnson
date: '2023-08-09 14:35:23 +0530'
category: Homelab
img: /assets/img/posts/2023-08-09/feat-1.png
keywords: HomelabUpdate, OPNSense, Syslog
permalink: /blog/homelab-update-opnsense-splunk-syslog/
usemathjax: true
imgdate: 2023-08-09
---

### Preface:
As part of the Homelab revamp, I decided it best to completely blow away all of the VM's and start fresh. This also included starting fresh with the Firewall/Router (Switching from pfSense to OPNSense) and documenting everything from day 1. The only thing which remains is the NAS (OpenMediaVault) and it's Docker setup, and that is because it's still serving a purpose as the home's "core" NAS and media server. Even the hypervisor it was running on was updated, and the drives which used to be passthrough via RDM were changed, however the data and OS remained. (Side note: While RDM works, my motherboard has 10 SATA ports which has two controllers that I can passthrough individually with some simple modifications to the ESXI passthru config, and is much simpler/cleaner IMO - i.e. I can change the PCI passthrough at a wim via GUI vs having to remap the RDM via CLI). 


![New Homelab, who dis?](/assets/img/posts/{{page.imgdate}}/1.png){:data-align="center"}

#### Splunk... Again? Why not (insert X logging/metrics solution)
Yes, Splunk, again. Reason? 

I freaking love Splunk (and I am studying for Splunk Certs, I currently use it at my job, etc). 

This time I've set it up to have a "Hot" storage and a "Cold" storage using "Magic"

![Magic](/assets/img/posts/{{page.imgdate}}/2.jpg){:data-align="center"}

Ok, it's not magic, but this time I decided it best to reinstall RHEL8 using CIS Benchmarks Level 1 Server for hardening (RHEL does this easily during the initial install, just separate the /home, /tmp, and /var/tmp onto sep partitions if memory serves right). I also added a 500 GB storage from one of the HDD's on one of the nodes in order to store "cold" log files (after 30 days) and keeping "hot" files on a separate 20GB partition. These have been mounted on /mnt/splunk (hot) and /mnt/splunk/cold (cold) in a nested fashion, which makes it easier in the Splunk GUI to assign hot/cold DB's. The Splunk Config has also been modified to default to these directories, CHMOD splunk:splunk on the appropriate folders, formatted, etc (not necessarily in that order).

![Data Partition Layout](/assets/img/posts/{{page.imgdate}}/3.png){:data-align="center"}

For those asking why not Graylog, ELK/Loki/Grafana/Promtail, etc., well those will probably be installed later on once everything else comes into play and integrated. We are rebuilding here.

Setting up Splunk on RHEL is straightforward and there are plenty of guides out there, so they won't be listed here.

#### Syslog Server... But Why?
While for a Homelab i could easily send all of my data directly into a Splunk Server, there are plenty of reasons to use a more Centralized Syslog Server and attached a Universal Forwarder to it to send data into Splunk. Plenty of whitepapers, even from Splunk, outlining this reason. My main reason? If I have to restart Splunk, I do not want to wait 10 minutes and lose 10 minutes of logs (potentially) if I want to update or install an app. Also expandability (maybe I want to run the aforementioned Graylog, ELK/Loki/Grafana/Promtail, etc.) in the future when I want to use those same logs for data later on.

Luckily setting this up is also rather straightforward/easy to do and can be done in under 10 minutes, barring however long it takes for you to pull down the ISO for your Linux of choice.

I am going to use RHEL 8 again, CIS Benchmarks Level 1 Server, and go ahead and run all the updates/upgrades prior to installing.

```bash
$sudo yum update
$sudo yum upgrade
```
Next we need to load the appropriate modules for TCP and UDP. This allows rsyslog to receive UDP connections. Open using your favorite text editor the following file:
/etc/rsyslog.conf. I'm going to use everyone's favorite, nano.

Then we are going to add/edit/uncomment the following lines to reflect the following:

```bash
module(load="imudp")
module(load="imtcp")
```
The remaining set up is mostly set up in accordance with this guide found here. I could re-type this, but I pretty much followed this guide to a T (and it was part of the splunk OPNSense Add On found on Splunks Apps webpage):

[OPNSense Syslog](https://splunk-opnsense-ta.ztsplunker.com/guides/guide-syslog/)

NOTE: When using other logging/dashboard solutions, File Permissions may need to be adjusted in the future, however for the time being this works and I'm sticking with it. I'll probably regret it later.

Important Note: When following this guide above, ensure you have added at least the group "splunk":
```bash
$sudo groupadd splunk
```

and added your user and the root account to that group if you follow this guide

```bash
$sudo usermod -aG splunk root
$sudo usermod -aG splunk justin
```
then check your settings at the end

```bash
$sudo systmectl status rsyslog
```
Now lets test OPNSense to ensure we are able to send it's logs to our new syslog server. In OPNSense -> System -> Settings -> Logging/targets -> click the "+".

I'm going to use TCP transport and for Applications leave blank to select everything, Keep the "Levels" at the default values, Facilities leave blank (to select everything) and enter the appropriate Hostname and port (514) and rfc5424 unchecked. Enter a description if you wish. Then click "save" and "apply".

![OPNSense Logging/target Screenshot](/assets/img/posts/{{page.imgdate}}/4.png){:data-align="center"}

Then back to the syslog server check the /var/log/remote folder for your log files:

![New Log Files](/assets/img/posts/{{page.imgdate}}/5.png){:data-align="center"}

### Splunk Setup
#### Setting up Splunk Index for OPNSense Logs
Now it's time to create a new Index to accept OPNSense Logs. Some people opt to have Indexes (indices?) based on function (Operating System, Networks, etc), some based on Softwares (Windows, Linux, Firewalls/Networks, etc.). I'm going for the latter.

In Splunk, I'll create an index for Network, which will capture everything for Firewall and Networks since my homelab is smaller and doesn't have a typical business separated Firewall, Router, Network Switch, etc.

Please note - I have a separate Cold Index set up, you may or may not have this set up.

![Splunk Index](/assets/img/posts/{{page.imgdate}}/6.png){:data-align="center"}

#### Side Note
If you do not have the Common Information Model Splunk Add on installed, now is a good time to get this installed and turned on.... Then accelerate the appropriate index:
[Common Information Model Acceleration - OPNSense](https://splunk-opnsense.ztsplunker.com/getting-started/configure/configure-dma/#enable-acceleration)


#### Connecting the Syslog Server to Splunk - Splunk Forwarder
Assuming everything up to this point has worked, now it's time to set up the Splunk Forwarder to capture these logs from the Syslog Server.

```bash
$ wget -O splunkforwarder-9.1.0.1-77f73c9edb85.x86_64.rpm "https://download.splunk.com/products/universalforwarder/releases/9.1.0.1/linux/splunkforwarder-9.1.0.1-77f73c9edb85.x86_64.rpm"
$ sudo -i splunkforwarder-9.1.0.1-77f73c9edb85.x86_64.rpm
```
then
```bash
cd /opt/splunk/bin
sudo ./splunk start --accept-license
```

Enter administrator username and password of your choosing.

Then enable boot start
```bash
$ sudo ./splunk stop
$ sudo ./splunk enable boot-start
```

then set up your inputs.conf and outputs.conf in /opt/splunkforwarder/etc/system/local/ as needed

#### Checking Splunk for Data
![It works](/assets/img/posts/{{page.imgdate}}/7.png){:data-align="center"}
If all goes well, you will have data in splunk. From here follow the guides for the OPNsense app for Splunk
[OPNsense Guide - The rest of the steps](https://splunk-opnsense.ztsplunker.com/getting-started/configure/configure-macros/)

#### Log Rotate (Optional - See Note)
The builtin package of logrotate is an amazing feature which can be used to "dump" log files after a certain period of time and can be easily set up by modifying the default configuration file at /etc/logrotate.conf and creating a new logrotate file:

```bash
$nano /etc/logrotate.d/opnsense
```
```bash
/var/log/remote/opnsense/*/*.log
{
  rotate 7
  daily
  missingok
  create 0640 root splunk
  notifempty
  compress
  createolddir 750 root splunk
  olddir /var/log/remote/old
  dateext
  dateformat -%Y-%m-%d
  dateyesterday
  sharedscripts
  postrotate
          /bin/kill -HUP `cat /var/run/syslogd.pid 2> /dev/null` 2> /dev/null || true
  endscript
}
```
Redhat.com explains this in more detail:
[Setting up logrotate in Linix](https://www.redhat.com/sysadmin/setting-logrotate)

However, Splunk also has an add-on (app) that can also auto-delete log files retrieved from a syslog server found here:

[Splunk Deleter](https://splunkbase.splunk.com/app/5356)