---
date: 2020-10-10 12:26:40
layout: post
title: HackTheBox - Cache
description: Medium Linux Machine containing Memcached Server and Docker Privilege Escalation
image: https://miro.medium.com/max/700/0*pEzAPWVI5YTjWcEs
optimized_image: https://miro.medium.com/max/700/0*pEzAPWVI5YTjWcEs
category: HackTheBox
tags:
  - docker
  - memcache
  - linux
author: anishka
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">anishka</a>. Also join me on discord.

The IP of this box is **10.10.10.188**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/670/1*IBCorClbNwU_02IHegdnVA.png "Large example image")

We have only 2 Open Ports, one for SSH and another for Web, so we directly move onto the web

## Web

Running the IP in the browser, we get a webpage related to hacking and also it tells the DNS for the box so we add it to our hosts file

![placeholder](https://miro.medium.com/max/1000/1*V7_brsLVGHexJHZuzud2Sw.png "Large example image")

Clicking on the Login option, we get redirected to the login page

![placeholder](https://miro.medium.com/max/1000/1*4ouc4W7bqnyQlrk41f_jFQ.png "Large example image")

Trying incorrect credentials leads us to alerts for both username and password separately

![placeholder](https://miro.medium.com/max/351/1*IoKZ_LLRWKb03xxIiRdtMg.png "Large example image")

![placeholder](https://miro.medium.com/max/420/1*ChQbsBZvttPaK8Cq2PbqZA.png "Large example image")

There was also an Author option which redirects us to **author.html** page

![placeholder](https://miro.medium.com/max/1000/1*rVg0yW8QttkuVlA0asf8qw.png "Large example image")

Here we see the name of the creator as ***Ash***, this might be the username we want

![placeholder](https://miro.medium.com/max/455/1*K3YhuT_d6qZtQa_s-vKiKw.png "Large example image")

Trying the username ***ash*** on the login shows its a correct username as we don’t get alerted for incorrect username like before

![placeholder](https://miro.medium.com/max/455/1*v8wRoXQ7jT3FWhoaMc3Y-g.png "Large example image")

Running Gobuster against the target gives us few results

![placeholder](https://miro.medium.com/max/233/1*fJqTY8uWXmKp225NShx-5g.png "Large example image")

We also see a ***/jquery*** directory which seems to be interesting

![placeholder](https://miro.medium.com/max/491/1*d42e89d85zwdBDhAoNHc9w.png "Large example image")

Inside the directory there is a ***functionality.js*** file which we see

![placeholder](https://miro.medium.com/max/495/1*Q2LgGtKgwVLATfdBrOfqzw.png "Large example image")

Here we see two functions validating the username and password , we see that user ***ash*** was correct and also we have a password too and we also know why we were getting alerts in incorrect *user/pass*

![placeholder](https://miro.medium.com/max/412/1*SiA_lzzcgVJ-HBIbLy0Uwg.png "Large example image")

Login in with the correct credentials redirects to ***net.html*** was already accessible even without login, so this login page was a rabbit hole

![placeholder](https://miro.medium.com/max/1000/1*iG5PxBilgxEKxGCbV4bwUA.png "Large example image")

But we can keep the credentials for future use as we weren’t able to login through **SSH** with this credentials

Since on the author page we saw that he has created some **Hospital Management System (HMS)**, I fuzzed for vhosts but no luck , but then randomly guessed another domain by the name ***hms.htb*** which leads use to different place

![placeholder](https://miro.medium.com/max/700/1*oIdkO9NuzrVjiVZnwfTRmQ.png "Large example image")

We tried the credentials we got before but no luck, running gobuster against this domain

![placeholder](https://miro.medium.com/max/229/1*Z3G0FReqIXecxTB1lO78NA.png "Large example image")

We got alot of results from it, one of them interestingly looks ***/portal***

![placeholder](https://miro.medium.com/max/1000/1*kZZxOrVNIKQjJZhNNSyUbg.png "Large example image")

We got another login page, this time for **OpenEMR** and sadly those credentials didn't worked here too, clicking on the **Register** option redirects to registration page

![placeholder](https://miro.medium.com/max/1000/1*KSjhY2zXL5CD549nviTswg.png "Large example image")

From the reported vulnerability of OpenEMR, we see that we can bypass the login authentication from the registration page , details of which is in the link on the end of this writeup

![placeholder](https://miro.medium.com/max/1000/1*oqFgEnV7V6ngQmV3F6gGgg.png "Large example image")

The above page is vulnerable to ***SQLi*** , so we use ***SQLMap*** against it to dump useful data

![placeholder](https://miro.medium.com/max/1000/1*kbqNMMgXRCf7uH_CF9SGrg.png "Large example image")

We got a username and encypted **bcrypt** password hash

![placeholder](https://miro.medium.com/max/700/1*wkmcBrCBpEElxUTd664wFQ.png "Large example image")

I used hashcat to crack the hash using mode **3200** and with ***rockyou.txt*** password list

![placeholder](https://miro.medium.com/max/700/1*q3fEMrAoy03hM61ZDSv5IA.png "Large example image")

We got the password cracked it was lame , was just ‘**xxxxxx**’

![placeholder](https://miro.medium.com/max/1000/1*Nfzc6nBd2QXiC_ltqRSRtQ.png "Large example image")

We used the credentials in the main OpenEMR login page and got accessed

![placeholder](https://miro.medium.com/max/1000/1*AhLsMidN-8CwapkXiPDQjw.png "Large example image")

Now we have a unrestricted file upload vulnerability where we upload a php web shell without any issues

![placeholder](https://miro.medium.com/max/700/1*k2ejltmjddpyvj7cdVUnqw.png "Large example image")

We got our shell working and now moving onto getting reverse shell on netcat

![placeholder](https://miro.medium.com/max/563/1*X-2-DI0cmjiIYSP6qWTYeQ.png "Large example image")

We got shell as ***www-data***, now we try to use the creds for user ash which we got before and then get the user flag

![placeholder](https://miro.medium.com/max/474/1*7eHbM620xw-VNoW5TNo5Hg.png "Large example image")

# Privilege Escalation

Running ***PSPY*** on the machine , we see that ***memcached*** is running on port **11211**

![placeholder](https://miro.medium.com/max/700/1*lFIWp8LFKC7euKVQppE4ag.png "Large example image")

We use telnet to connect to the port locally and then get important dumps

![placeholder](https://miro.medium.com/max/185/1*vG8i1epcjNiOkXkuy9q_3g.png "Large example image")

We got credentials for user ***luffy*** and then login into that user

![placeholder](https://miro.medium.com/max/204/1*hlawbbfKfijLDTKK98Nwfg.png "Large example image")

Checking the groups of the current user, we see that user ***luffy*** is a group member of ***docker***

![placeholder](https://miro.medium.com/max/460/1*XuC1JPKCgJGWUanq-ZzoHA.png "Large example image")

We see that a docker image for ubuntu is installed on the docker and we can mount the ***/root*** directory to the ***/mnt*** directory and get the root flag

![placeholder](https://miro.medium.com/max/626/1*gyIpD_vTNlZbe1zvNmNk7g.png "Large example image")


# References

<a href="https://www.hackingarticles.in/penetration-testing-on-memcached-server/">Pentesting Memcached Server</a>

<a href="https://www.hackingarticles.in/docker-privilege-escalation/">Docker Privilege Escalation</a>

<a href="https://www.open-emr.org/wiki/images/1/11/Openemr_insecurity.pdf">OpenEMR - Project Insecurity</a>


<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










