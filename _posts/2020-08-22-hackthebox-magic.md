---
date: 2020-08-22 12:26:40
layout: post
title: HackTheBox - Magic
description: Medium Level Linux Machine
image: https://cdn-images-1.medium.com/max/800/0*AqLjA4WXH2kkuwmc
optimized_image: https://cdn-images-1.medium.com/max/800/0*AqLjA4WXH2kkuwmc
category: hackthebox
tags:
  - hacking
  - infosec
author: hulegu
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">hulegu</a>. Also join me on discord.

The IP of this box is **10.10.10.185**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/800/1*iVh22DO_FKL39y_fd7tm3w.png "Large example image")

We got only 2 Open Ports , one for SSH and other for Web

## Web

Running the IP in the browser

![placeholder](https://cdn-images-1.medium.com/max/1200/1*XHCQ_0w6fHluCTnuppgC2Q.png "Large example image")

We get a webpage with a lot of pictures, also on the down left corner we see a link to a **Login** Page

![placeholder](https://cdn-images-1.medium.com/max/1200/1*cNTq0Mj6BjchtP0ez0bmFw.png "Large example image")

We see we have a login option here, we try some **SQLi Login Bypass** techniques

![placeholder](https://cdn-images-1.medium.com/max/800/1*T9ctoT03fHRe7ZBRFIvlLw.png "Large example image")

The Payload we used for password field

> admin'or 1=1 or ''='

![placeholder](https://cdn-images-1.medium.com/max/1200/1*wbTsEdARcyPsKXIWQKvcPw.png "Large example image")

We see a file upload here , I tried multiple ways of bypassing the file upload restriction out of which one way worked for me

![placeholder](https://cdn-images-1.medium.com/max/800/1*uKbzjsn98TDjIIi7S1-2CQ.png "Large example image")

So we just change the metadata using exiftool and put out PHP web shell and then upload it

![placeholder](https://cdn-images-1.medium.com/max/800/1*lY-UiYN7WHRfUQcpa4h61w.png "Large example image")

After uploading the file, we get the message of the successful upload like above pic

Now we access the file which we cant get from the link source in the main page

![placeholder](https://cdn-images-1.medium.com/max/800/1*z31RlhIanu4rRP10EU-s7g.png "Large example image")

Now we try to execute command and see if it works

![placeholder](https://cdn-images-1.medium.com/max/800/1*qD26KiBBzvFM-E-baCj1bw.png "Large example image")

We got command execution successfully and now we move onto getting reverse shell and check the netcat listener

![placeholder](https://cdn-images-1.medium.com/max/800/1*70GKzXYltaRuki_OCqlY0A.png "Large example image")

We got shell as **www-data** and now we do our enumeration on the webroot directory

![placeholder](https://cdn-images-1.medium.com/max/800/1*55W7itCduybrA8C8dOrNwg.png "Large example image")

We see a ***db.php5*** file, looking at the code

![placeholder](https://cdn-images-1.medium.com/max/800/1*xHJk8BwcVR9iffaX6zVL3g.png "Large example image")

We see some credentials leading to some DB , when trying to use this for the user on the machine

![placeholder](https://cdn-images-1.medium.com/max/800/1*KI2crS_X6KTrMpB_7gIp2w.png "Large example image")

It fails, but we know that we had SQLi in the login page, we can copy the request and run ***SQLMAP*** against it

![placeholder](https://cdn-images-1.medium.com/max/800/1*64HZPag-lg90X9HI-M32aQ.png "Large example image")

We dumped credentials and now use the password for use ***theseus***

![placeholder](https://cdn-images-1.medium.com/max/800/1*FpQECGL6Q_ajDDLEQ-6vfA.png "Large example image")

We are now user ***theseus*** and now we get the user flag

![placeholder](https://cdn-images-1.medium.com/max/800/1*rBD-G-xSw_zWaKqrLruITw.png "Large example image")

## Privilege Escalation

Checking the groups of the current user, we see that it is the group member of group users

![placeholder](https://cdn-images-1.medium.com/max/800/1*MQAsp1GoUuqhWTBb7jsCDg.png "Large example image")

Checking any files or binaries with the group permissions of group ***users***

We see that **/bin/sysinfo** has group permissions for ***users***

![placeholder](https://cdn-images-1.medium.com/max/800/1*qDR_JhGCYWOFFB-Q6C45iQ.png "Large example image")

The binary wasn't any ordinary binary so I just ran ***ltrace*** onto it and saw that the binary runs ***lshw and fdisk*** to provide hardware info and disk info, since there is no path mentioned for those 2 programs, we can potentially path hijack and use it for our own benefits

![placeholder](https://cdn-images-1.medium.com/max/800/1*d0nJStUr22SaTFFAkQhCyA.png "Large example image")

Now we create our own binary named the same as the vulnerable one and then change the path so that our binary gets executed first instead of the original one

![placeholder](https://cdn-images-1.medium.com/max/800/1*hW8GKiWVrRvWjjbWPymcSQ.png "Large example image")

After running **sysinfo**, we get a shell spawned and now we are root

![placeholder](https://cdn-images-1.medium.com/max/800/1*7WxzdyaCXQnIxQLdjIFEFA.png "Large example image")

# References

<a href="https://pentestlab.blog/2012/12/24/sql-injection-authentication-bypass-cheat-sheet/">SQL Injection Authentication Bypass Cheat Sheet</a>
