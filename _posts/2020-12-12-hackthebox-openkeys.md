---
date: 2020-12-05 12:26:40
layout: post
title: HackTheBox - OpenKeys
description: Medium Level OpenBSD Machine containing an unique login bypass and openbsd local root exploitation
image: https://miro.medium.com/max/700/0*o_ITJMlU-bGB6TP6
optimized_image: https://miro.medium.com/max/700/0*o_ITJMlU-bGB6TP6
category: HackTheBox
tags:
  - openbsd
  - medium
author: feodore
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">feodore</a>. Also join me on discord.

The IP of this box is **10.10.10.199**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/691/1*Jg4SwX5WzPF11p4r38-MzA.png "Large example image")

We have only 2 Ports Open on full port scan , running SSH and Web

Moving towards the web part

## Way To User

Running the IP on the browser , it redirects us to a login page

![placeholder](https://miro.medium.com/max/1000/1*SwNA0szuMf2G98mNszt9jQ.png "Large example image")

I tried default creds, tried bruteforcing normal creds , nothing worked so went to run ***gobuster*** against it and got

![placeholder](https://miro.medium.com/max/334/1*Z1TQciACOgAUcF42M-wrLQ.png "Large example image")

Checking the ***/includes*** directory , we get

![placeholder](https://miro.medium.com/max/686/1*E5IRhtvdJYFVCo8dz_8BHQ.png "Large example image")

We see two files , one of them being a PHP file for authentication and other being its SWP file, checking the SWP file

![placeholder](https://miro.medium.com/max/1000/1*fB-N2b0yGVhiuDCppzhukg.png "Large example image")

We see that there is a domain named ***jenniferopenkeys.htb*** which I will put in my hosts file and then get this SWP to my local machine and try to recover the actualy code

![placeholder](https://miro.medium.com/max/449/1*idj1g_2ovEQwIrms8jfL4A.png "Large example image")

We see the PHP code and from here we see that it is executing something named ***check_auth*** from the web directory

![placeholder](https://miro.medium.com/max/654/1*Vu-rthrNU1N6oyrKXrwaqQ.png "Large example image")

Trying to access the directory, we see that we have the ***check_auth*** file and we download it to our local machine

![placeholder](https://miro.medium.com/max/664/1*XACcTnX2KfCSFVM8Oeajrw.png "Large example image")

Checking the file type , we see its an ELF 64-bit binary

![placeholder](https://miro.medium.com/max/700/1*MJgU4PnwvKZgINVj49Qc-g.png "Large example image")

Running the ***strings*** commands, we see that there is something named **auth_userokay** , checking on google about it

![placeholder](https://miro.medium.com/max/298/1*0D8nHZ1kJzStwlBZQ55n8w.png "Large example image")

Checking on Google about the exploit for this, OpenBSD has authenticated based CVE where we can bypass the login but putting **-schallenge** in username section

![placeholder](https://miro.medium.com/max/700/1*YT24HkK0oSnyhVE0W1ExFg.png "Large example image")

We bypassed the login authentication and got redirected to a file named **sshkey.php**, it returns an error message that OpenSSH key was not found for the username we put, but from earlier we know that there is a potential user named “***jennifer***”

![placeholder](https://miro.medium.com/max/700/1*T5LYjxItuBljypgAI5sd_Q.png "Large example image")

Also if we checked the swp file on the web , it takes usernames by **$_REQUEST**, we can put the username in the cookies section as shown below

![placeholder](https://miro.medium.com/max/690/1*PTyfDTQNEYE5D_KuMQej5g.png "Large example image")

Forwarding the request , we get the SSH key for user ***jennifer***

![placeholder](https://miro.medium.com/max/700/1*AWLo1FoQktwEyo66ii-neQ.png "Large example image")

Now we copy this to our local machine and then connect to user ***jennifer*** through SSH

![placeholder](https://miro.medium.com/max/700/1*TutU5iBF_Ip3zu6_qlajyw.png "Large example image")

We got user , time for priv esc


## Way To Root

Running the ***uname -a*** command , we see that the current machine is **OpenBSD 6.6**

![placeholder](https://miro.medium.com/max/397/1*G6L2K2TmjIClRlbC0qDWtA.png "Large example image")

Checking for exploits for this one, I got a ***xlock*** exploit which had a bash script which I used below

![placeholder](https://miro.medium.com/max/700/1*OiF33dDS4usoH67oYG49eA.png "Large example image")

We got root and also the root flag




# References

<a href="https://www.secpod.com/blog/openbsd-authentication-bypass-and-local-privilege-escalation-vulnerabilities/">OpenBSD Authentication Bypass and Local Privilege Escalation Vulnerabilities</a>

<a href="https://github.com/bcoles/local-exploits/blob/master/CVE-2019-19520/openbsd-authroot">OpenBSD AuthRoot Exploit</a>




<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










