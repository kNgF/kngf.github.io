---
date: 2020-04-26 12:26:40
layout: post
title: HackTheBox - Control
subtitle: HackTheBox Control Walkthrough by Ferllen
description: Control runs a vulnerable PHP web application that controls access to the admin page by checking the X-Forwarded-For HTTP header. By adding the X-Forwarded-For HTTP header with the right IP address we can access the admin page and exploit an SQL injection to write a webshell and get RCE. After pivoting to another user with the credentials found in the MySQL database, we get SYSTEM access by modifying an existing service configuration from the registry. 
image: https://miro.medium.com/max/810/0*VVgNDHYF1kz8pziK
optimized_image: https://miro.medium.com/max/810/0*VVgNDHYF1kz8pziK
category: HackTheBox
tags:
  - ctf
  - infosec
  - hacking
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a>. Also join me on discord.

The IP of this box is **10.10.10.167**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/573/1*PBjKRxFLzWk3b2Fdwo3NeA.png "Large example image")

Running service scans against the open ports

![placeholder](https://miro.medium.com/max/810/1*YTyrUYDlE4hI-jpwL2HX6w.png "Large example image")

We see HTTP , msrpc and mysql services running on the open ports , moving to the web part

## Web Part

Running the IP in the browser

![placeholder](https://miro.medium.com/max/573/1*znXUj8Vjwg90-9N5dsK-ag.png "Large example image")

The webpage looks very good, running Gobuster against it

![placeholder](https://miro.medium.com/max/810/1*W0z6I1I2pcjq30Jf7gSfKg.png "Large example image")

We get alot of results, before checking them , we check the source code of the main webpage

![placeholder](https://miro.medium.com/max/573/1*BqolBlaj_TVfx-UgcIKCCw.png "Large example image")

We see something interesting in the comment section which has some to do tasks which includes certificate location to **\\192.168.4.28\myfiles**

From the Gobuster scan, we saw admin.php page so we try to access the page

![placeholder](https://miro.medium.com/max/573/1*pnSwvS0dd7nhkj4Fwm36xg.png "Large example image")

We get access denied error message, also it tells us to go through the proxy to access the page

We intercept the request in Burp Suite and then send the request to Repeater

![placeholder](https://miro.medium.com/max/810/1*RDQY6pguOCjSIrKZiCbkAg.png "Large example image")

Here we now add a **X-Forwarded-For** header with the value of the IP address which we got from the source code comments back before

![placeholder](https://miro.medium.com/max/810/1*6qtfQMovzuIUZR7v7lfHbQ.png "Large example image")

We sent the request and saw that this time we got accessed to the admin.php page, so we just go the proxy settings

![placeholder](https://miro.medium.com/max/810/1*jCksez1zZ6a0MCXQ62_bow.png "Large example image")

We add the required header so that whenever we request the admin.php or any other page, it redirects it through the proxy automatically

So now we access the page

![placeholder](https://miro.medium.com/max/810/1*7fIX2HXnS-OO4gyV7U20_Q.png "Large example image")

We see something related to products, also there is a Search field which we use

![placeholder](https://miro.medium.com/max/810/1*pUyTa7dlxOY6AXCilozgdQ.png "Large example image")

We see that it queries something, so we just copy the request to a file and then run sqlmap against it

![placeholder](https://miro.medium.com/max/810/1*-Zq7bdu2pRUYI89Tozha8Q.png "Large example image")
![placeholder](https://miro.medium.com/max/810/1*5Nk83PYj1D7y4zbKN9qFvA.png "Large example image")

We see that page is vulnerable to SQL Injection, down below we can confirm the database version and the databases

![placeholder](https://miro.medium.com/max/459/1*GYc8EdqS1rV8LoqX9SXeVg.png "Large example image")

We now move onto dumping users and passwords

![placeholder](https://miro.medium.com/max/810/1*tnuiEMOzKoP2kVUwdcg5JQ.png "Large example image")

Now we crack these encrypted passwords using sqlmap’s password cracker

![placeholder](https://miro.medium.com/max/573/1*BeOIL64qvCYQFzpXhlbt5A.png "Large example image")

We cracked the password successfully, but we didnt found anyway of getting in as WinRM and SSH both were closed

![placeholder](https://miro.medium.com/max/810/1*v3jclkAvZZCinuJRk5cauA.png "Large example image")

We uploaded a php webshell through sqlmap and then try to access it

![placeholder](https://miro.medium.com/max/494/1*F9SrX3uzOL-RRu8hciy6nw.png "Large example image")

We got shell successfully and now we upload netcat through sqlmap too and then try to get reverse shell to work more properly

![placeholder](https://miro.medium.com/max/810/1*rHvcBqekcgJPu8_YGMCsoA.png "Large example image")

Checking back the netcat listener

![placeholder](https://miro.medium.com/max/494/1*qZFAdZf6T_2Z4FWN4EVYnA.png "Large example image")

We got reverse shell and now we escalate to ***hector*** user

![placeholder](https://miro.medium.com/max/573/1*HgR7RI6201pm6KC9Z_DfXQ.png "Large example image")

We got shell as ***hector*** but we have a limited shell where we cant get any response of any command , so we use netcat again to get reverse shell through this escalated user shell we got

![placeholder](https://miro.medium.com/max/573/1*dBIfgqRKjdc60hGYjYJ9-A.png "Large example image")

Checking the netcat listener

![placeholder](https://miro.medium.com/max/494/1*skaMIjisI1YJXLQviWeiOg.png "Large example image")

We got proper shell as hector user, moving further to get the user flag which is usually stored under the Desktop folder of the user

![placeholder](https://miro.medium.com/max/494/1*D9N420qDwFKJQcGHZCt1Ng.png "Large example image")

Now time for privilege escalation to root

## Privilege Escalation

Checking for powershell history commands

![placeholder](https://miro.medium.com/max/573/1*Uw12KPGfGnM6_3IoYy0WJw.png "Large example image")

We see that two powershell commands were used for registries so we use the below command to get the services which user Hector has FullControl with

`get-acl HKLM:\System\CurrentControlSet\services\* | Format-List * | findstr /i "hector Users Path Everyone"`

![placeholder](https://miro.medium.com/max/810/1*pKH6cdcpJcrzRcBTgebHow.png "Large example image")

Since we see alot of services, we use wuauserv service which is **Windows Updater** service

![placeholder](https://miro.medium.com/max/810/1*_I0crVKCf4nyVtommAdeGQ.png "Large example image")

Here we hijacked the service and changed it to run the netcat for us

![placeholder](https://miro.medium.com/max/494/1*UUobjJ4vpPN-AxQacQF6Wg.png "Large example image")

After starting the service and checking back to the netcat listener

![placeholder](https://miro.medium.com/max/494/1*7us7pUKeufCulxiUVMI3Kg.png "Large example image")

We got shell as ***nt authority\system***

# Youtube Video

 <iframe width="420" height="315" src="https://www.youtube.com/embed/_ynBlMqLWqU?autoplay=1"></iframe> 

# References

<a href="https://renenyffenegger.ch/notes/Windows/registry/tree/HKEY_LOCAL_MACHINE/System/CurrentControlSet/index">Registry: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet</a>

<a href="https://www.absolomb.com/2018-01-26-Windows-Privilege-Escalation-Guide/">Windows Privilege Escalation Guide</a>

<a href="https://book.hacktricks.xyz/windows/windows-local-privilege-escalation">Windows Local Privilege Escalation</a>









