---
date: 2020-11-07 12:26:40
layout: post
title: HackTheBox - Tabby
description: Easy Linux Machine where we exploit LFI to get tomcat creds and then exploit tomcat manager using WAR files and then LXD group permission exploitation for privilege escalation
image: https://cdn-images-1.medium.com/max/800/0*iwEoJcs6HrQdcoE4
optimized_image: https://cdn-images-1.medium.com/max/800/0*iwEoJcs6HrQdcoE4
category: HackTheBox
tags:
  - linux
  - easy
author: anishka
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">anishka</a>. Also join me on discord.

The IP of this box is **10.10.10.194**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/1200/1*POkPpNmRee585UnFMvirYw.png "Large example image")

We have 3 Ports Open , Port 22 for SSH , Port 80 and Port 8080 for Web running Apache httpd and Apache Tomcat

Moving directly to the web part

## Web

Running the IP in the browser, we see

![placeholder](https://cdn-images-1.medium.com/max/1200/1*SHPUI2QeB5Vw2k6LjqpMwg.png "Large example image")

We get a web page related to hosting, hovering over the **News** field , we get a domain ***megahosting.htb*** which I put onto my hosts file and then click it

![placeholder](https://cdn-images-1.medium.com/max/1200/1*jVZltPdjTG_JssdpMkR1nA.png "Large example image")

Looking into the URL, we see it has **file=statement** , potentially it is include a file from the server, so trying LFI payload for ***/etc/passwd*** leak

![placeholder](https://cdn-images-1.medium.com/max/1200/1*HxPPaV3Yqwzjt4YlrQD64Q.png "Large example image")

We confirm that we have **LFI**, but since it is in unreadable form, so we can just see the source code for readable form

![placeholder](https://cdn-images-1.medium.com/max/800/1*FOSab11n9Jbl8pxJAJ8MQQ.png "Large example image")

I tried to find some interesting stuffs onto the server, but no luck

Since we had **Apache Tomcat** running on Port 8080, we jump into that now

![placeholder](https://cdn-images-1.medium.com/max/1200/1*cITESw4uWEXFlgOeeY0pxw.png "Large example image")

On the main home page, we get the default ***It Works!*** page which tells us the version of the Apache Tomcat and also to access the manager or host-manager webapp , we need to do the basic authentication, but we dont have any creds

Since on the main web page, we saw that Apache Tomcat is installed under ***/usr/share/tomcat9***, upon googling I came to know that tomcat9 creds are stored under ***/usr/share/tomcat9/etc/tomcat-users.xml***

So we can use the LFI to pull out the file

![placeholder](https://cdn-images-1.medium.com/max/800/1*qZWirkrtKjwu7BAWzyUIqQ.png "Large example image")

We got the creds, now trying to login into the ***host-manager*** webapp

![placeholder](https://cdn-images-1.medium.com/max/800/1*YI05Pc_dor1FW6qhRzMBzw.png "Large example image")

We put our creds and login in

![placeholder](https://cdn-images-1.medium.com/max/1200/1*OtY3GVQ6V253t7SlopwsBQ.png "Large example image")

We got logged in successfully, but we cant upload WAR file to get reverse shell from here , for that we need to move into the ***manager*** webapp

![placeholder](https://cdn-images-1.medium.com/max/1200/1*8jMvvgSZKAndSuK-rYHDKQ.png "Large example image")

We get Access denied on ***manager*** webapp , trying the text version of it

![placeholder](https://cdn-images-1.medium.com/max/800/1*zluaECS_yZ1HA9pCCxQQYw.png "Large example image")

We do not get restrict on the text version, so we can now create our malicious WAR file using ***msfvenom***

![placeholder](https://cdn-images-1.medium.com/max/800/1*qaOqaQS4z0xtT61SXReK7g.png "Large example image")

We created our malicious WAR file and now we are ready to deploy

![placeholder](https://cdn-images-1.medium.com/max/800/1*F8xrtH0IWtxSMyFZDHhy-A.png "Large example image")

We use curl to deploy our WAR file into the server and we can confirm that our file has been uploaded

![placeholder](https://cdn-images-1.medium.com/max/800/1*4gThFBVP0oUZK1rq9huQzA.png "Large example image")

Now we spawn ***msfconsole*** and set our multi handler options

![placeholder](https://cdn-images-1.medium.com/max/800/1*7MxFLnFyTsNwaDmvqWt3PA.png "Large example image")

Now we just trigger the WAR file from the web

![placeholder](https://cdn-images-1.medium.com/max/800/1*OuA7nzGkLgCz2RB78e-Izw.png "Large example image")

Looking onto the msfconsole

![placeholder](https://cdn-images-1.medium.com/max/800/1*BLix2YA1-bxgO4uKiZPMAw.png "Large example image")

We got shell session successfully and now we get a proper shell

![placeholder](https://cdn-images-1.medium.com/max/800/1*j-h9FjE4JlXHKyg9Uexzsw.png "Large example image")

Digging into the webroot directory, we found a zip file which is password protected, so we get that zip file to our local machine and crack the password using ***fcrackzip***

![placeholder](https://cdn-images-1.medium.com/max/800/1*QoHKVN661_o4-fEkSx9apA.png "Large example image")

We cracked the password and now we unzip the file

![placeholder](https://cdn-images-1.medium.com/max/800/1*ZA-y3V_fB_XYoCSOfP-ZYA.png "Large example image")

Looking into each files and folders, we see that we don't have anything interesting

![placeholder](https://cdn-images-1.medium.com/max/800/1*EaEa2UmXjTZt5PH1DhwrPQ.png "Large example image")

Using the same password for user ***ash*** , we logged into his account

![placeholder](https://cdn-images-1.medium.com/max/800/1*MXAbH7Z_hj_tdGR-ycAesw.png "Large example image")

Time to get the user flag

![placeholder](https://cdn-images-1.medium.com/max/800/1*nZNGRPu4zPsiKtyWw5KYUg.png "Large example image")

## Privilege Escalation

Checking the group permissions

![placeholder](https://cdn-images-1.medium.com/max/800/1*mAErTSOh8lJdqBFgUdZRvA.png "Large example image")

User ***ash*** is a group member of ***lxd*** group which has a potential privilege escalation vector

![placeholder](https://cdn-images-1.medium.com/max/800/1*ynTebrxZ9_xGic_um5vAwg.png "Large example image")

Here we initialize everything for our lxc image, to confirm we can check the network created for ***lxc***

![placeholder](https://cdn-images-1.medium.com/max/800/1*nukUcyXcGnY2dTXbzJACRg.png "Large example image")

Now we import the image we created from ***lxd-alpine-builder***

![placeholder](https://cdn-images-1.medium.com/max/800/1*aetC8Kep6mq8R13lTBRVWw.png "Large example image")

Our image has been imported and now we do the following commands to get our task done finally

![placeholder](https://cdn-images-1.medium.com/max/800/1*rWp_f4o1akQg4CcL1Gg0bQ.png "Large example image")

We got shell and we can confirm we are root now , to access the main root directory, we will have to move to ***/mnt/root/root*** and get the root flag

![placeholder](https://cdn-images-1.medium.com/max/800/1*GRI6ShBbmUqzSmP6rKr4lA.png "Large example image")




# References

<a href="https://packages.debian.org/sid/all/tomcat9/filelist">Tomcat Package List</a>

<a href="https://www.hackingarticles.in/lxd-privilege-escalation/">LXD Privilege Escalation</a>

<a href="https://github.com/saghul/lxd-alpine-builder">LXD Alpine Image Builder</a>



<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










