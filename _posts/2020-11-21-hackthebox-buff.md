---
date: 2020-11-21 12:26:40
layout: post
title: HackTheBox - Buff
description: Easy Windows Machine with exploiting public vulnerabilities for both user and system 
image: https://cdn-images-1.medium.com/max/800/0*RQVwAZrSGGYCjV_8
optimized_image: https://cdn-images-1.medium.com/max/800/0*RQVwAZrSGGYCjV_8
category: HackTheBox
tags:
  - windows
  - easy
author: feodore
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">feodore</a>. Also join me on discord.

The IP of this box is **10.10.10.198**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/800/1*eIHlcBToEC9aPzZHIk_r6Q.png "Large example image")

We got only 2 Open Ports , moving directly to the web

## Web

Running the IP in the browser

![placeholder](https://cdn-images-1.medium.com/max/1200/1*LeWZDL0yc4JYt9ByE761sQ.png "Large example image")

We see it is a nice website , checking each and every link

![placeholder](https://cdn-images-1.medium.com/max/800/1*ouettj0_9dE0BX6ssHXa3A.png "Large example image")

On the Contact page, we see that it tells us that the website is made using ***Gym Management Software 1.0***, upon looking in the internet we get a public exploit for it which we use here

![placeholder](https://cdn-images-1.medium.com/max/800/1*urM_JQDovCZCUyHIKPfe1A.png "Large example image")

We get a shell which is actually unstable, so we get stable shell after uploading netcat and using it

![placeholder](https://cdn-images-1.medium.com/max/800/1*jTXPDHXnKpcIFqyZdWkb8w.png "Large example image")

## Privilege Escalation

Running ***winPEAS***, we see that there is a binary named **CloudMe_1112.exe** which is actually the binary for CloudMe application version 1.11.2

![placeholder](https://cdn-images-1.medium.com/max/800/1*K1PwGbr8N-ovIJYi8q8cmA.png "Large example image")

Upon looking more, we see that port **8888** is open and listening locally which might be the **CloudMe** service running so we port forward it to our local machine

![placeholder](https://cdn-images-1.medium.com/max/800/1*on-gBQjJLlp5re_qAtQQsQ.png "Large example image")

Now there is a public exploit for ***CloudMe 1.11.2*** on **Exploit-DB** which we will use here but before that we need to edit our shellcode there

![placeholder](https://cdn-images-1.medium.com/max/800/1*lCJJGB1DpIElyFY9K8b_rw.png "Large example image")

We copy the shellcode above and then overwrite it in the exploit

![placeholder](https://cdn-images-1.medium.com/max/800/1*Se6W6SVfjJMTjBbgjc5-JQ.png "Large example image")

We now run the exploit and check the netcat listener

![placeholder](https://cdn-images-1.medium.com/max/800/1*039LFOZFM_hxwd_sye4Yow.png "Large example image")

We got root!


# References

<a href="https://www.exploit-db.com/exploits/48506">Gym Management System 1.0 - Unauthenticated Remote Code Execution Exploit</a>

<a href="https://www.exploit-db.com/exploits/48389">
CloudMe 1.11.2 - Buffer Overflow (PoC)
</a>




<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










