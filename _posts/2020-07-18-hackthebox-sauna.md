---
date: 2020-07-18 12:00:00
layout: post
title: HackTheBox - Sauna
description: Sauna is an easy Active Directory based machine
image: https://miro.medium.com/max/573/0*0TdT46pOT_Ut05iA
optimized_image: https://miro.medium.com/max/573/0*0TdT46pOT_Ut05iA
tags:
  - hacking
  - pentesting
  - active directory
author: hulegu
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">hulegu</a> . Also join me on discord.

The IP of this box is ***10.10.10.175***

# Port Scan

Running NMAP full port scan on it , we get

![placeholder](https://miro.medium.com/max/810/1*uWKLi2IF8BKJRjoZ4EV7eA.png "Large example image")

We see alot of Open Ports, also from the Ports Open we see that this is yet another Active Directory box

Moving further to the web part

# Web Part

Checking the web part on the browser

![placeholder](https://miro.medium.com/max/810/1*vJl2zLI2T0KG_go0UVHJxw.png "Large example image")

Checking the “***About Us***” section

![placeholder](https://miro.medium.com/max/810/1*12qDPJLDvyeeyflZc3N7CA.png "Large example image")

Usually from my experiences from solving AD based machines, the users are saved in the format of “**First Letter of the First Name**” with “**Last Name**”

![placeholder](https://miro.medium.com/max/202/1*AiwodSe7pyf2g50YHqjUWA.png "Large example image")

We save the usernames in the file named ***users***

Now we run an Impacket tool named “***GetNPUsers.py***”

![placeholder](https://miro.medium.com/max/810/1*UFiql2Gx4sd9pz7jGxsZsg.png "Large example image")

We got a Kerberos session hash for user ***fsmith*** which we will crack using ***john***

![placeholder](https://miro.medium.com/max/810/1*ZxkhTfGr6WXXFH-sSHLUQg.png "Large example image")

We cracked the password for user ***fsmith*** successfully

We use ***Evil-WinRM*** to get the user shell

![placeholder](https://miro.medium.com/max/573/1*HcjsyTsf5b0P9ADkpuwwew.png "Large example image")

Moving further to privilege escalation

# Privilege Escalation

We check Registry for User Autologon

![placeholder](https://miro.medium.com/max/810/1*K2ryoz8Hzc6KjMJYknAaRw.png "Large example image")

We got password for user ***svc_loanmanager***

![placeholder](https://miro.medium.com/max/573/1*qb7ey_iGOoTgVaF2DYxONw.png "Large example image")

We have ***svc_loanmanager*** user as ***svc_loanmgr*** here, so we use ***Evil-WinRM*** again to connect to the user

![placeholder](https://miro.medium.com/max/573/1*vy5YsMDy5Ce3cU-O6ez7Iw.png "Large example image")

We now upload **SharpHound.ps1** script to the box and then run

![placeholder](https://miro.medium.com/max/810/1*iD963GaR3hqpujl3-F6lTQ.png "Large example image")

We collection data for bloodhound and now will download the zip file containing the data

![placeholder](https://miro.medium.com/max/810/1*eXAyTpkh--1amjfxjzsf6Q.png "Large example image")

Since Evil-WinRM is full of functionalities, it provides us with a download option too

We first start our ***neo4j*** console

![placeholder](https://miro.medium.com/max/573/1*uOQjPlQPMxL8tV5KCaRrxw.png "Large example image")

Now we log through the browser

![placeholder](https://miro.medium.com/max/573/1*PxI9GNpQuv-iNtinNi2znA.png "Large example image")

We connected and now get the bolt address on the bloodhound

![placeholder](https://miro.medium.com/max/810/1*IvrPb5VW3sTVhBPmqcfqNQ.png "Large example image")

Running Bloodhound with the address and creds we got

![placeholder](https://miro.medium.com/max/573/1*9a1CinU4oEbvfY1RKX-kaQ.png "Large example image")

We dragged the zip file we got post SharpHound and then see that the current user has **DCSync** rights

![placeholder](https://miro.medium.com/max/810/1*JvBaavCEBGflXhvn3Atlmg.png "Large example image")

We now use ***secretsdump*** from **impacket** to dump the hashes

![placeholder](https://miro.medium.com/max/810/1*KJC3Dw0OUOYLi207gJA-Gg.png "Large example image")

We dumped the hashes of ***Administrator*** and now use it with **wmiexec** from impacket to get a shell as **Administrator**

![placeholder](https://miro.medium.com/max/810/1*qK2knJOlFpTyNMYH-m9Eew.png "Large example image")

We got shell as Administrator and now move into getting the root flag

![placeholder](https://miro.medium.com/max/460/1*cjmxrS1hkxdiqZ0mvSBDgg.png "Large example image")


<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 
