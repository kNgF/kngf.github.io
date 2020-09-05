---
date: 2020-09-05 12:26:40
layout: post
title: HackTheBox - Remote
description: Easy Active Directory Based Machine
image: https://cdn-images-1.medium.com/max/800/0*7s78AjhTEzgI5qgF
optimized_image: https://cdn-images-1.medium.com/max/800/0*7s78AjhTEzgI5qgF
category: HackTheBox
tags:
  - active directory
  - windows
author: anishka
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">anishka</a>. Also join me on discord.

The IP of this box is **10.10.10.180**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/800/1*0Gg_DKJMqmT1o-3rD2uDBw.png "Large example image")

We see a whole loads of Open Ports , but one of the interesting one looks Port 2049 which is running NFS mountd service, so we just move onto it first and see any mounts accessible to us

![placeholder](https://cdn-images-1.medium.com/max/800/1*FyBIfGm8z2ON1Wv9fheuIQ.png "Large example image")

We see **/site_backups** which is accessible to everyone, so we just mount it to our tmp folder

![placeholder](https://cdn-images-1.medium.com/max/1200/1*Nzg59lK8ZMZtknA5uGKUTA.png "Large example image")

We see there are a lot of stuffs here , but before that we just move onto the web and see what we have there

![placeholder](https://cdn-images-1.medium.com/max/1200/1*TwcdAK0DfRzRSVpJDIsYGw.png "Large example image")

We have a cool page, running Gobuster against it

![placeholder](https://cdn-images-1.medium.com/max/800/1*2sgCzobdVbJ306pou-UBHQ.png "Large example image")

We get a big result out of which we see **/Install** , moving towards it

![placeholder](https://cdn-images-1.medium.com/max/1200/1*qkwaoRCbgUWCfcs8hPRbKA.png "Large example image")

We get redirected to a login page and if we see clearly, it is running **Umbraco CMS**, looking for potential exploits on ***searchsploit***

![placeholder](https://cdn-images-1.medium.com/max/1200/1*Vz-Gev-n6Q-uXk9UOLF6ug.png "Large example image")

We see we have an exploit which will help us getting RCE but it requires authentication, so we now move towards the mounted NFS we did before

![placeholder](https://cdn-images-1.medium.com/max/800/1*mkrE0sFSm7221Hb0vsJcNA.png "Large example image")

Now looking at **Umbraco.sdf** file using ***strings***

![placeholder](https://cdn-images-1.medium.com/max/1200/1*UrnrJ-V5y3sdOevmtFTnIA.png "Large example image")

We see usernames and encrypted password, so I take the hash for **admin@htb.local** username and will crack as the algorithm used here is ***SHA1***

![placeholder](https://cdn-images-1.medium.com/max/1200/1*CRr3vsEgODesK2DLpL7HDA.png "Large example image")

We cracked the password ***"baconandcheese"***

![placeholder](https://cdn-images-1.medium.com/max/1200/1*2789gXVNWviPhJaa3Y6Hxg.png "Large example image")

We got successfully logged in with the cracked password for that username, now moving towards the exploit where we have to do some modifications

![placeholder](https://cdn-images-1.medium.com/max/800/1*bVurNZ_PqNMcUVHUW0l-jw.png "Large example image")

We here first try to get pinged back first

![placeholder](https://cdn-images-1.medium.com/max/800/1*qORZxJ3SffKZ1-RpUUGmkQ.png "Large example image")

We got pinged and so we are ready to get reverse shell

![placeholder](https://cdn-images-1.medium.com/max/1200/1*oqbVUmg8-1HhC18E0Kaj5g.png "Large example image")

Looking onto the netcat listener

![placeholder](https://cdn-images-1.medium.com/max/800/1*_39_iSzI1DSLM8AVcXjUjA.png "Large example image")

We got shell and now moving towards getting the user flag

![placeholder](https://cdn-images-1.medium.com/max/800/1*sN4Y_8lEZWcR1Wak1BW7gQ.png "Large example image")

Time for Privilege Escalation

## Privilege Escalation

Running PowerUp.ps1 powershell priv esc script, we get

![placeholder](https://cdn-images-1.medium.com/max/800/1*c2UwDrlFAbEhuVAHIumlKg.png "Large example image")

We see we have a vulnerable service named ***UsoSvc*** which we can use to abuse and get root

![placeholder](https://cdn-images-1.medium.com/max/1200/1*RUEYdbBhk3l2uGwvXeBwJQ.png "Large example image")

We are good to go and see the ***netcat*** listener

![placeholder](https://cdn-images-1.medium.com/max/800/1*EQLIpe0nc2iBQZni-67AIA.png "Large example image")

We got shell as **NT Authority\system** and now we get the root flag

![placeholder](https://cdn-images-1.medium.com/max/800/1*bpJTcpZaBEXPG2Ig2JfoXA.png "Large example image")


# References

<a href="https://www.exploit-db.com/exploits/46153">Umbraco CMS Exploit</a>

<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md">Windows Privilege Escalation - PayloadAllTheThings</a>


<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










