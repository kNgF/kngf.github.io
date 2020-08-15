---
date: 2020-08-15 12:00:00
layout: post
title: HackTheBox - Traceback
description: Easy Linux Box
image: https://miro.medium.com/max/700/0*he0s6ifHwUTcmnHN
optimized_image: https://miro.medium.com/max/700/0*he0s6ifHwUTcmnHN
category: HackTheBox
tags:
  - hackthebox
  - linux
author: huleguogedei
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">hulegu</a>. Also join me on discord.

The IP of this box is **10.10.10.180**

# Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/700/1*kmhFS_QpSd383JJ-7J0cUQ.png "Large example image")

We only see 2 Open Ports, Port 22 and Port 80 for SSH and Web, we directly move towards the web part

# Port 80 — Web

Opening the IP in the browser

![placeholder](https://miro.medium.com/max/700/1*c2BBpczYZizrpq1_o0iBhA.png "Large example image")

We see that there is a message that the page has been owned by someone or pwned. Also it says that the attacker has left a backdoor and this page seems to be defaces.

Checking the page source

![placeholder](https://miro.medium.com/max/694/1*qC5SOKZysRlW1gV2LPYumw.png "Large example image")

We see that there is a comment that says “***Some of the best web shells that you might need***”, which might be indicating hint where to look for , so searching on Google for that , we come across some of the best web shells down below

![placeholder](https://miro.medium.com/max/700/1*lTaE94N62PlSEtOTlKHfpQ.png "Large example image")

We see that we have alot of web shells here, so we try to find each one of them into the web directory

![placeholder](https://miro.medium.com/max/1000/1*Xljl10eUYxQQPyte0ZSWyg.png "Large example image")

After testing each one of them, we see that ***smevk.php*** web shell was accessible and it had username password login at the startup

![placeholder](https://miro.medium.com/max/700/1*9OFNVhZ86GoGQV3UNBa8_Q.png "Large example image")

Looking at the source code of the web shell, we see that the username and password both were ***admin***

![placeholder](https://miro.medium.com/max/1000/1*CEDDjUEV17rWd3-g_omIzQ.png "Large example image")

After login, we see that we can upload our files to the web directory, since this shell wasnt comfortable to me, I just uploaded my php shell and got code execution from there

![placeholder](https://miro.medium.com/max/700/1*gYRjJA0Icc3MVTxaZEewbQ.png "Large example image")

Now we try to get reverse shell

![placeholder](https://miro.medium.com/max/700/1*sAs5JaJ0KAxJc6JR-EqvYA.png "Large example image")

We got reverse shell successfully, now we check the contents of the home directory of the current user

![placeholder](https://miro.medium.com/max/662/1*0g42eAJo4hCgQxov_u05pQ.png "Large example image")

We see a **note.txt** file, checking the contents of it

![placeholder](https://miro.medium.com/max/483/1*SWkSyyAiM0Rv06TKEwiViQ.png "Large example image")

We see that the sysadmin user has left a tool to practice **Lua** and we have to find it, before that we just try to do ***sudo -l*** and see if we can use sudo without password

![placeholder](https://miro.medium.com/max/700/1*Vlbv2smn-XOr2QoZ95kISA.png "Large example image")

We see that we can use sudo without password on user sysadmin for **/home/sysadmin/luvit**, **Luvit** is the tool which is used to practise Lua

![placeholder](https://miro.medium.com/max/1000/1*2DzCCsUkcFQk_k8-PjUcaw.png "Large example image")

We created a Lua one liner script which will help us get reverse shell and then we run the script through Luvit so that we can get our reverse shell as ***sysadmin***

![placeholder](https://miro.medium.com/max/508/1*c3vt2s5ZLz5Ut7p9H8Slrg.png "Large example image")

We got reverse shell as ***Sysadmin*** user successfully and now moving onto getting user flag

![placeholder](https://miro.medium.com/max/371/1*lLd4fcRGsq0OM5xeOvCwJQ.png "Large example image")

# Privilege Escalation

Running ***Linpeas.sh*** script, we see

![placeholder](https://miro.medium.com/max/700/1*C-JxoUnf7rmcHJDYCg49iw.png "Large example image")

We see that we have Group Writable directory which is **/etc/update-motd.d/**

Upon looking on the man page of **update-motd**, we see that

![placeholder](https://miro.medium.com/max/700/1*-GnOUSskqHGu4b422CfJzQ.png "Large example image")

So we just have to edit a script in that directory and just make a login to trigger it, so first I will put my own ssh keys on the ssh folder of ***webadmin*** user since it was only writable to us

![placeholder](https://miro.medium.com/max/700/1*SFmvPwmKQsFdWxg6MDDtkw.png "Large example image")

Now we add our bash one liner reverse shell command in on of the script in that folder

![placeholder](https://miro.medium.com/max/1000/1*1unoJdB1DyGIxvPzry6IDQ.png "Large example image")

Now we try to login to webadmin user through SSH

![placeholder](https://miro.medium.com/max/629/1*P4CpOrTIJfGSfkTa4h7zZQ.png "Large example image")

Looking back to the netcat listener

![placeholder](https://miro.medium.com/max/530/1*YjNn_30mXeAtCCRhoteuZg.png "Large example image")

We got shell as root successfully and moving onto getting root the flag

![placeholder](https://miro.medium.com/max/215/1*awsAIB8iWTexIw8d7QMZ0w.png "Large example image")

# References 

<a href="http://manpages.ubuntu.com/manpages/trusty/man5/update-motd.5.html">update-motd</a>

<a href="https://github.com/TheBinitGhimire/Web-Shells">Web Shells List</a>

<a href="https://gtfobins.github.io/gtfobins/lua/">Lua Sudoer Exploit</a>

<a href="https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS">LinPEAS</a>

<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 
