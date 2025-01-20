---
layout: post
title:  "VSCode Servers"
summary: "How to remotely edit code files"
author: justinmjohnson
date: '2022-07-20 14:35:23 +0530'
category: VSCode
img: /assets/img/posts/2022-07-20/feat-1.png
keywords: VSCode
permalink: /blog/VSCODE-Server/
usemathjax: true
imgdate: 2022-07-20
---

{%- comment -%} Please delete below and place your page content here {%- endcomment -%}

If you are like me, you work on a laptop, a desktop, a work laptop, sometimes you are out and about and may not have your devices with you (you may even be borrowing your in-laws computers), and you may need to make a slight adjustment to some sort of code. Maybe you forgot to turn on one service, or portion of service to make something work while away from home, you can VPN to your home, but not necessarily SSH to your desktop. What if you could set up a centralized “coding server” that could access all of your servers and their code, or be able to access your github/gitea/source control repositories.

Well you can, and it runs in a web browser. VS Code has [Code Server](https://code.visualstudio.com/blogs/2022/07/07/vscode-server)

```bash
$ curl -fsSL https://code-server.dev/install.sh | sh
```

Then we need to edit our config to set our default password to log into the server, as well as set the “bind-addr” to 0.0.0.0 (so that we can access it from outside of the “local machine”. This is so we don’t have to access it only from the local machine, however if you are using a reverse proxy set up on the local machine leave this be (if you are using a reverse proxy on a different virtual machine/IP set this to 0.0.0.0:8080.

```bash
$ sudo nano ~/.config/code-server/
```

![Afterward ctrl + x to save](/assets/img/posts/{{page.imgdate}}/2.png){:data-align="center"}

Afterward ctrl + x to save

Then lets enable the server for our user:

```bash
$ sudo systemctl enable --now code-server@USER
```
Where USER is your username. Then check on the status to make sure its running

```bash
$ sudo systemctl status code-server@USER
```
![It's running](/assets/img/posts/{{page.imgdate}}/3.png){:data-align="center"}

If all is well you should be able to access it from the IP address of your virtual machine with port 8080 (in my instance its 10.10.10.21:8080. Later (not covered in this blog) I will put it behind a reverse proxy in order to access it from the outside world, coupled with some sort of SSO authentication such as Authelia or Authentik.


![Login Screen](/assets/img/posts/{{page.imgdate}}/4.png){:data-align="center"}

Use the password we set up in our Config and now we should be able to access our server.


![Code Server Screen](/assets/img/posts/{{page.imgdate}}/5.png){:data-align="center"}

Now this isn’t using SSL (https), which is why it is recommended to use a reverse proxy with an SSL certificate in order to protect yourself if you expose this to the outside world. From here you can set up your github/self hosted git/version control repository for your code. You can also set up your extensions that you would normally use (for your programming languages, etc.) and it will carry over each time you sign in, no matter what device you log in from. This saves time from having to install VSCode and setting everything up (granted the single-sign on through Microsoft or GitHub does carry over most everything).

Some initial drawbacks I’ve seen:
1) Multi-user support is not available at this time, so its not a “true” server solution so to speak.
2) Some keystrokes that you are normally used to don’t carry over without some setting tweaks. Just don’t try to hit “Ctrl+w” to close a tab when you think you are closing a “File Tab”… Instead you will close your Code-Server window.
3) You can’t use your desktop VSCode to remote into the Code-Server

So if you need a code server environment where you can log into via browser or have a use case (such as using a tablet when away) code-server may be right for you. Either way this is a cool project that has potential.