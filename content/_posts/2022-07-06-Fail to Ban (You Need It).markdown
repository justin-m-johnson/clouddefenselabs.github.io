---
layout: post
title:  "Fail to Ban - You need it!"
summary: "Easy to use, quick to deploy, another layer of safety"
author: justinmjohnson
date: '2022-07-06 14:35:23 +0530'
category: Fail2Ban
img: /assets/img/posts/2022-07-06/feat-1.png
keywords: Fail2Ban, SSH, Cybersecurity
permalink: /blog/fail2ban/
usemathjax: true
imgdate: 2022-07-06
---

{%- comment -%} Please delete below and place your page content here {%- endcomment -%}

Fail2Ban has been around for a long time and is one of the most used tools today when it comes to protecting web-facing servers. It’s simple, yet many #homelab users can easily overlook this simple to utilize tool when running linux webservers.

So what is Fail2Ban? From their [website](https://www.fail2ban.org):
![From fail2ban.org - July 2022](/assets/img/posts/{{page.imgdate}}/feat-1.png){:data-align="center"}

Pretty simple. It’s an added layer of protection to your website, but as it clearly states, it doesn’t replace weak authentication or lack of two factor or pub/priv keys. Let’s get started on setting up Fail2Ban.

I primarily use Ubuntu/Debian based (Apt Package Manager) operating systems, so adjust accordingly if you use RHEL/CentOS/Fedora (DNF/Yum), OpenSuse (Zypper or Yast), or Arch (btw) (*snicker*).

First lets update/upgrade out system just for good measure then we’ll install Fail2Ban

```bash
$ sudo apt update && sudo apt upgrade
$ sudo apt install fail2ban
```

![Screenshot of install](/assets/img/posts/{{page.imgdate}}/2.png){:data-align="center"}

Next we will enable and start Fail2Ban

```bash
$ sudo systemctl enable fail2ban
$ sudo systemctl start fail2ban
```

![Enabled](/assets/img/posts/{{page.imgdate}}/3.png){:data-align="center"}

Now we copy our default Fail2Ban jail.conf file over to a jail.local file. The reason for this is because when Fail2Ban updates, the jail.conf file will be overwritten, and any changes we make in it will therefore be overwritten.

```bash
$ sudo cp /etc/fail2ban/jail.{conf,local}
```
Fail2Ban utilizes Jails, which are “Filters” and “Actions”. Filters are what Fail2Ban “watches” and Actions are what Fail2Ban does when the filters are met. To set up our Fail2Ban we create our jail file using our favorite editor, nano (sue me):

```bash
$ sudo nano /etc/fail2ban/jail.local
```
Look for “ignoreip”, uncomment it (if it is commented it with a “#” and add any/all IP Addresses that you want to ensure that NO lockouts happen (such as your local network).

![ignoreip uncommented - take out the #](/assets/img/posts/{{page.imgdate}}/4.png){:data-align="center"}

There are other settings you can adjust here as well, such as “bantime” (how long an IP address is banned for), “findtime” (the duration of the “offense” filtered) and “maxretry” (how many times until Fail2Ban takes action). Adjust these to your liking.

Afterwards exit out of your editor and restart Fail2Ban:

```bash
$ sudo systemctl restart fail2ban
```
So now we will set up our “jail” file in the “/etc/fail2ban/jail.d” directory. Let’s create one for SSH:

```bash
$ sudo touch /etc/fail2ban/jail.d/sshd.local
```

Next we will enter our configuration

```md
[sshd]
enabled = true
port = ssh
action = iptables-multiport
maxretry = 3
bantime = 600
```
This creates an “sshd” jail, enables it, identifies the port (ssh, you can also list the port number(s) here to listen on), action (what action it will take, defined in /etc/fail2ban/action.d), the maxretry and bantime should be self-explanatory. Another line can be added called “logpath” that it will watch a certain log file/path for failed tries.

Afterwards, restart Fail2Ban again:

```bash
$ sudo systemctl restart fail2ban
```

```bash
To check your settings, you can “tail” your Fail2Ban log and see for yourself
```

![Tailing fail2ban](/assets/img/posts/{{page.imgdate}}/5.png){:data-align="center"}

I’ll test some failed SSH attempts (I’ll remove my local subnet from the jail.local file created above)

![Testing out some settings](/assets/img/posts/{{page.imgdate}}/6.png){:data-align="center"}

And while I was testing this, I forgot that I was already testing this in another window and successfully locked myself out of my machine….

![Whoops](/assets/img/posts/{{page.imgdate}}/7.png){:data-align="center"}


Sigh… I’ll be able to log back into that machine eventually…

Just kidding, I’ unlock myself now (through my ESXi server). Let me show you the commands:

```bash
$ sudo fail2ban-client set sshd unbanip 10.10.10.151
$ tail /var/log/fail2ban.log
```

![Banned and Unbanned](/assets/img/posts/{{page.imgdate}}/8.png){:data-align="center"}
Success!

![Unbanned myself](/assets/img/posts/{{page.imgdate}}/9.png){:data-align="center"}