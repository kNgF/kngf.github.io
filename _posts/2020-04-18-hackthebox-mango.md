---
date: 2020-04-18 12:26:40
layout: post
title: HackTheBox - Mango
description: Mango was a medium box with a NoSQSL injection in the login page that allows us to retrieve the username and password. The credentials we retrieve through the injection can be used to SSH to the box. For privilege escalation, the jjs tool has the SUID bit set so we can run scripts as root. 
image: https://miro.medium.com/max/573/0*2J9dM0alSBSRE9NH
optimized_image: https://miro.medium.com/max/573/0*2J9dM0alSBSRE9NH
category: HackTheBox
tags:
  - ctf
  - infosec
  - hacking
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a>. Also join me on discord.

The IP of this box is **10.10.10.162**

## Port Scan

Running masscan on it , we get

`masscan -p1-65535,U:1-65535 10.10.10.162 --rate=1000 -e tun0`

![placeholder](https://miro.medium.com/max/573/1*mUjA65Bh3h2LDZAT_OsIQg.png "Large example image")

Only 3 Open Ports were discovered, running NMAP against them

![placeholder](https://miro.medium.com/max/810/1*qpK_Vcwq91Zkv-RlPQGndQ.png "Large example image")

Since these two are common ports for SSH and Web, moving further to the Web Part

## Port 80 — Web

Running the IP in the browser

![placeholder](https://miro.medium.com/max/472/1*t-nSuGiyklu6-538v9tBMA.png "Large example image")

Returned with a forbidden error , moving to the other web port, i.e, 443

![placeholder](https://miro.medium.com/max/573/1*Rp8gGgjRpW4v-u0ZZZL8nQ.png "Large example image")

Viewing the certificate

![placeholder](https://miro.medium.com/max/573/1*9ZYu8lYX5OHvfyEePgZU8A.png "Large example image")

Adding the VHOST onto our hosts file

![placeholder](https://miro.medium.com/max/573/1*_dwKhtF5LWkiWs65H9UAvA.png "Large example image")

Also now accepting the certificate

![placeholder](https://miro.medium.com/max/810/1*Oe4NpnBcvfaxhHKQI7SrQg.png "Large example image")

We get a replica of Google Search Engine with the name Mango

Moving on the ***staging-order.mango.htb*** vhost

![placeholder](https://miro.medium.com/max/810/1*1FJ14m1cJ4B_IxcjSFxGLg.png "Large example image")

We get a login page , testing for NoSQL injection we first intercept the request through Burp Suite

![placeholder](https://miro.medium.com/max/494/1*saA7mfqY0iAo_WtimV_RmA.png "Large example image")

Putting the [$ne] (not equal) string just after the username and password paramter and sending the request

![placeholder](https://miro.medium.com/max/494/1*NbV2P_4z3mSp-RKtiZ0cyA.png "Large example image")

The request looks like above and now we check the response

![placeholder](https://miro.medium.com/max/810/1*K_IZRG5glfhmlKKIfXZO0g.png "Large example image")

We see some different result this time than when we were trying to login with random common creds , also we get redirected to home.php page which means we bypassed the authentication page. So we now try to extract data through NoSQL injection

Now we use a python script to automate the data extraction stuff for this

![placeholder](https://miro.medium.com/max/573/1*xYBEbBkNk1dJxerMrKvctA.png "Large example image")

Running this script

![placeholder](https://miro.medium.com/max/439/1*sqO3dFt6MZvifbC8CoFy-A.png "Large example image")

We got the password for ***admin*** , now changing the script and looking for the password for ***mango*** user

![placeholder](https://miro.medium.com/max/439/1*YgDgFMRzql_U5XNctrrLOg.png "Large example image")

We got the password , note that we have to omit the dollar sign($)

Trying to connect to ***mango*** user through SSH

![placeholder](https://miro.medium.com/max/810/1*36k920_tC_U8KhHDhZ_eQw.png "Large example image")

When we try to connect to ***Admin*** user through SSH, it failed and also we can confirm that there is an ***Admin*** user on the box

![placeholder](https://miro.medium.com/max/573/1*rq2AdTqJMETfRlm1Zanjjg.png "Large example image")

Also, the user flag is located in the home folder of the ***Admin*** user which cant access

![placeholder](https://miro.medium.com/max/339/1*l_jvLOY8HQS-M789byxW2Q.png "Large example image")

So we have our admin user creds, so we directly use **su** to login

![placeholder](https://miro.medium.com/max/339/1*nO7qD89PqzhONheK1LgdiA.png "Large example image")

We logged in successfully, now lets get the user flag

![placeholder](https://miro.medium.com/max/358/1*vqY_r7ktGh6BXwaxmCovnA.png "Large example image")

Now moving onto the priv esc part

## Privilege Escalation

Running the traditional **LinEnum.sh** script

![placeholder](https://miro.medium.com/max/573/1*PuGt6slhtOJG05mc4vtJcg.png "Large example image")

We get an interesting SUID file which has permissions for groups on Admin user and we can exploit this to get root

For now, I just run the binary to read the root flag

![placeholder](https://miro.medium.com/max/573/1*GtW0__YOWwe_s6T--ymjPw.png "Large example image")

Here we get an error as jjs takes the root.txt as java file but in the end returns the error exposing the root flag

For the shell method , you can watch my video or live stream on <a href="fb.gg/arkanoidgg">Facebook</a> or <a href="youtube.com/c/ArkanoidGaming">YouTube</a>

# Youtube Video

 <iframe width="420" height="315" src="https://www.youtube.com/embed/srbAOVuCMfw?autoplay=1"></iframe> 
 
# References

<a href="https://gtfobins.github.io/gtfobins/jjs/">jjs | GTFOBins</a>

<a href="https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection">PayloadAllTheThings</a>








