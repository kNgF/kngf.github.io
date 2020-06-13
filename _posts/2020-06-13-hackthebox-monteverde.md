---
date: 2020-06-13 13:24:49
layout: post
title: HackTheBox - Monteverde
description: Medium Level Active Directory Box
image: https://cdn-images-1.medium.com/max/716/0*H4zBo3vRXpnsXAh6
optimized_image: https://cdn-images-1.medium.com/max/716/0*H4zBo3vRXpnsXAh6
category: HackTheBox
tags:
  - Active Directory
  - Windows
  - Pentesting
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="">ferllen</a>. Also join me on discord.

The IP of this box is 10.10.10.172

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/716/1*4ktZvy0SZxz50GvbRSmcww.png "Large example image")

We got alot of Open Ports, running services scan against them

![placeholder](https://cdn-images-1.medium.com/max/1075/1*OzsuHyjaCWXIvvljAyMc2A.png "Large example image")

We see many services running, also Active Directory on this box so we now move onto running ***enum4linux*** tool

![placeholder](https://cdn-images-1.medium.com/max/1075/1*h8-1y_7MeGJ1WhMJLNApHw.png "Large example image")

We get alot of users from the tool so we save it in a file named users

Now moving onto running a metasploit module ***smb_login*** which checks for valid smb login creds

![placeholder](https://cdn-images-1.medium.com/max/1075/1*zG2YgeNlH4UTimDFvfUe-g.png "Large example image")

Here we set the username list and password same as the users we got from the enum4linux tool and then run the module

![placeholder](https://cdn-images-1.medium.com/max/716/1*rTcNtefS415L6ABGAOKCAg.png "Large example image")

We can see that user **SABatchJobs:SABatchJobs** is a valid credential

Using these credentials with ***smbclient***, we see alot of open shares

![placeholder](https://cdn-images-1.medium.com/max/716/1*oRw0RFRAZ6SgK2_Xvl7s9A.png "Large example image")

We see a share named ***"users$"*** so we connect to it

![placeholder](https://cdn-images-1.medium.com/max/716/1*_5NM2a2hkOCQVTxgxoRA1A.png "Large example image")

We connected to ***"users$"*** share through SMB successfully

![placeholder](https://cdn-images-1.medium.com/max/716/1*mzs0bX7tpDNbKF6XdoJPvQ.png "Large example image")

We have few user folders here, upon looking on every folders, we see something interesting in **mhope** folder

![placeholder](https://cdn-images-1.medium.com/max/716/1*atl2ssgzqr8LKbyBb97oEA.png "Large example image")

We see a file named **azure.xml** , so download it to our box and see the contents

![placeholder](https://cdn-images-1.medium.com/max/716/1*HYOVD2pCZ-ZFjjlbNXEk9A.png "Large example image")

We see a password, since **WinRM** port was open on the box , so I try to connect through ***Evil-WinRM*** with multiple users we got on the box

![placeholder](https://cdn-images-1.medium.com/max/1075/1*KlRo42tYbT2dgdIjgc_CSQ.png "Large example image")

We got connected with user ***mhope*** successfully, moving onto getting the user flag which is usually located in the **Desktop** folder

![placeholder](https://cdn-images-1.medium.com/max/716/1*U_QCifDamtecu5jt2xwAcQ.png "Large example image")

Moving further to privilege escalation

## Privilege Escalation

Running the ***whoami /all*** command, we get

![placeholder](https://cdn-images-1.medium.com/max/1075/1*zrjHBaIP4mKFXntugBPQOQ.png "Large example image")

We see that the current user has group permissions of ***MEGABANK\Azure Admins***

Upon looking much on google for Azure Hacking , we come to know many things about **Azure AD Connect**

![placeholder](https://cdn-images-1.medium.com/max/1075/1*QiJ6WLviWxYy5iEMRveSsQ.png "Large example image")

As we have **Microsoft SQL Server** , so we run ***SQLCMD*** to get the databases and we get few databases upon which **ADSync** is the one which we are interested in

![placeholder](https://cdn-images-1.medium.com/max/1075/1*rA53P3jowzHfMCIu5TiUXw.png "Large example image")

The above commands fetched the **administrator**'s password and gave us the decrypted form of it

Connecting with ***Evil-WinRM*** through these creds

![placeholder](https://cdn-images-1.medium.com/max/716/1*48Q9OkUzTDOZ3GrKuXOFJQ.png "Large example image")

We got connected successfully, moving onto get the root flag

![placeholder](https://cdn-images-1.medium.com/max/716/1*qOc0l_CkvpDpDJTdWygdfQ.png "Large example image")

Its always fun to solve Windows AD boxes

# References

<a href="https://blog.xpnsec.com/azuread-connect-for-redteam/">Azure AD Connect for Red Teamers</a>


