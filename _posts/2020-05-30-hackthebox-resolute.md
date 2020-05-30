---
date: 2020-05-30 22:55:45
layout: post
title: HackTheBox - Resolute
description: Resolute is an easy Active Directory based machine
image: https://cdn-images-1.medium.com/max/1600/1*QrzMRz6kWJ9tvUkgfMi_Xg.png
optimized_image: https://cdn-images-1.medium.com/max/1600/1*QrzMRz6kWJ9tvUkgfMi_Xg.png
category: HackTheBox
tags:
  - hacking
  - ctf
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a> . Also join me on discord.
The IP of this box is 10.10.10.169

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/1600/1*NjvSAZ8Gg5b1lPiuA7O3qQ.png "Large example image")

We got alot of Open Ports, running service scan on it

![placeholder](https://cdn-images-1.medium.com/max/2400/1*KzsPsVT11rl783S8EPMCkw.png "Large example image")

We can see we have much things to enumerate on

### Enum4Linux

Using the Enum4Linux tool

![placeholder](https://cdn-images-1.medium.com/max/2400/1*6tP1nLJ9wwxVTpbcwpyrKw.png "Large example image")

We can see that user ***marko***'s password is exposed as ***Welcom123!***

We can use these creds for login through multiple methods, one of them being WinRM

![placeholder](https://cdn-images-1.medium.com/max/1600/1*F7aHTHSlbdXTrCiJ9gjpGg.png "Large example image")

From the NMAP scan, we see that Port **5985** is Open, so we can try **Evil-WinRM** tool to connect

![placeholder](https://cdn-images-1.medium.com/max/1600/1*uLeaaSmvXHXv573OJZ4nFQ.png "Large example image")

We get authentication error, but from the *enum4linux* tool, we got many users and now we put those users in a *txt* file

![placeholder](https://cdn-images-1.medium.com/max/1600/1*8PCT6-f9GtcKHVdXfdebjg.png "Large example image")

Here we have list of the users in a txt file named *users.txt*

![placeholder](https://cdn-images-1.medium.com/max/1600/1*iSZW2hajRHYKvoLyKvView.png "Large example image")

Here we use simple bash scripting to bruteforce users and now we wait till we get the correct creds and get logged in automatically

![placeholder](https://cdn-images-1.medium.com/max/1600/1*ITdHR78jbT0WYXdo6XoPvw.png "Large example image")

We got connected successfully as user ***melanie***, moving onto getting user flag

![placeholder](https://cdn-images-1.medium.com/max/1600/1*s8YnWQAHrfqpNAOudzwdUw.png "Large example image")

Moving further to priv esc

### Privilege Escalation

Checking for hidden files and folders in the root directory

![placeholder](https://cdn-images-1.medium.com/max/1600/1*W2oNQF-dP8-kqwO_18O-sQ.png "Large example image")

We see a strange folder named **PSTranscripts**, entering it we dont see anything until looking for hidden files and folders again

![placeholder](https://cdn-images-1.medium.com/max/1600/1*43pRM8iJHaPrVYrAWb0H8w.png "Large example image")

We see one more folder in it and looking further into it

![placeholder](https://cdn-images-1.medium.com/max/1600/1*MKRbxc2LZS7XNjv0iKdynQ.png "Large example image")

We get a txt file related to Powershell or something so we check the contents of it

![placeholder](https://cdn-images-1.medium.com/max/2400/1*tWsR_TBnToGiidJ0IeIsXw.png "Large example image")

If we look carefully, we can see that it leaks password for user ***ryan*** so we again use **Evil-WinRM** to connect to that account

![placeholder](https://cdn-images-1.medium.com/max/1600/1*lBAVmHfnXtIO2-VJRxy3hw.png "Large example image")

We got connected successfully, looking for the group membership of the current user

![placeholder](https://cdn-images-1.medium.com/max/2400/1*fUHxo6BE3VnkDinTlWEurg.png "Large example image")

We can see that the current user is a group member of ***DnsAdmins*** which is prone to a getting **SYSTEM using DLL injection** method

![placeholder](https://cdn-images-1.medium.com/max/2400/1*15TfSk6lR7TbVVP9jPERQQ.png "Large example image")

We create a malicious dll using ***msfvenom*** for the dll injection

![placeholder](https://cdn-images-1.medium.com/max/2400/1*T01DM91Km3i2VBAMohBnCA.png "Large example image")

Also we start up a smbserver using ***Impacket's smbserver.py***

![placeholder](https://cdn-images-1.medium.com/max/1600/1*k0KmEKoP2SuLILkmg1A2iQ.png "Large example image")

Now we injected the malicious dll and then check the netcat listener

![placeholder](https://cdn-images-1.medium.com/max/1600/1*Ck63xMCDF4VkzYheQweIBA.png "Large example image")

We got shell as **SYSTEM** and now we get the root flag

![placeholder](https://cdn-images-1.medium.com/max/1600/1*AsfCB0sbRsipxrBn0-zpYA.png "Large example image")

# References

<a href="https://ired.team/offensive-security-experiments/active-directory-kerberos-abuse/from-dnsadmins-to-system-to-domain-compromise">Privilege Escalation Method</a>















