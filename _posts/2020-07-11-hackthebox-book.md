---
date: 2020-07-11 12:00:00
layout: post
title: HackTheBox - Book
category: HackTheBox
image: https://miro.medium.com/max/573/0*E85M9vyF7yDnQcV1
optimized_image: https://miro.medium.com/max/573/0*E85M9vyF7yDnQcV1
tags:
  - hacking
  - pentesting
author: hulegu
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">hulegu</a> . Also join me on discord.

The IP of this box is ***10.10.10.176***

# Port Scan

Running NMAP full port scan on it , we get

![placeholder](https://miro.medium.com/max/810/1*ksVZKW4yokd-BrY7g4NBTw.png "Large example image")

We just get only 2 Open Ports, one for SSH running on Port 22 and other for web running on Port 80

Moving to the web part

# Port 80 — Web

Running the IP on the browser, we get

![placeholder](https://miro.medium.com/max/573/1*hDThyt_Kczm30gTW7SteKQ.png "Large example image")

Trying default credentials, we don't get any luck so we move towards the Sign Up page

![placeholder](https://miro.medium.com/max/573/1*GlLPBSADR8baORtGCP8mmA.png "Large example image")

We now put our details to create an account

![placeholder](https://miro.medium.com/max/573/1*hfGv4evVs17kjSqSVuHvTQ.png "Large example image")

Now we just signup and then login through the created account

![placeholder](https://miro.medium.com/max/810/1*GA5qFj8Zd42J3t5E5CaQKw.png "Large example image")

After login, we see that we have a cool website , looking further more

![placeholder](https://miro.medium.com/max/573/1*w-mXUsY35NDcTe-06-TCpA.png "Large example image")

We have something in Contact Us page which leaked the admin email address

![placeholder](https://miro.medium.com/max/573/1*SqdbDB4U3opGVrZXPDK_CQ.png "Large example image")

We try for SQL Truncation Attack by intercepting the request in Burp

![placeholder](https://miro.medium.com/max/810/1*oR_UFAmbv9ChusZj9AXxSg.png "Large example image")

Now we forward the request and then try to login to the admin user

![placeholder](https://miro.medium.com/max/573/1*fnHsjg-X4z-A15lA62nLgg.png "Large example image")

Clicking on Sign In, we see

![placeholder](https://miro.medium.com/max/810/1*qGwmcroZkR9rFIhUrwugkQ.png "Large example image")

We don't see any differences here as its looks all the same as a normal user like we created before

![placeholder](https://miro.medium.com/max/810/1*WuYUosW7SrQJLD7WCQrqqQ.png "Large example image")

Running Gobuster scan, we see a useful directory named **/admin**

![placeholder](https://miro.medium.com/max/810/1*lu6l0OF9xbgotCaeablPGQ.png "Large example image")

Opening the link, we see that we have a different Admin Sign In Page created for admin

![placeholder](https://miro.medium.com/max/573/1*GubQcdOYTZZxnky_-FujVQ.png "Large example image")

So we now sign in using the credentials we created from SQL Truncation

![placeholder](https://miro.medium.com/max/810/1*oicWnOHicYpe99E9aNBREA.png "Large example image")

We logged in successfully as admin and we see completely different stuff from before

![placeholder](https://miro.medium.com/max/810/1*_g1LQTWVAcJ8s32BQULh8Q.png "Large example image")

Now moving back to the normal user section, and going to the Collections area, we see that we can upload a file with author name and book title

![placeholder](https://miro.medium.com/max/573/1*QBZU65aTu-ZmHM8jp7pKUA.png "Large example image")

So at first I try to upload a php webshell

![placeholder](https://miro.medium.com/max/477/1*4FWWctaEaXkaN90iiZSiBQ.png "Large example image")

We get a popup stating that the admin will evaluate the upload and update the list

![placeholder](https://miro.medium.com/max/810/1*MzmlqW5AVAXlDeoSoq7DYw.png "Large example image")

On the Admin page area, going to the collections section and downloading the both of the PDF

![placeholder](https://miro.medium.com/max/573/1*2x0UeHsziBsYLVqF9_jNWg.png "Large example image")

The users pdf file contains the list of Names and Emails of accounts created on the page

![placeholder](https://miro.medium.com/max/573/1*toChLCtGKSlm3uvbmO47BA.png "Large example image")

On the Collections PDF file, we see that our file which we uploaded shows here, also you notice that the name and author is also reflecting on the PDF , so we try some HTML Injections

![placeholder](https://miro.medium.com/max/573/1*Ub_H124_OMfxF2a2z3MntA.png "Large example image")

We try the to bold the author name and book title and see the results

![placeholder](https://miro.medium.com/max/573/1*oUfWRxFLxAxTXJiVoXEHZQ.png "Large example image")

We see that our HTML Injection worked, so we will now move into getting XSS which will help us getting local file reads, for which we will use the below command

```<script>x=new  XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///etc/passwd");x.send();</script>```

![placeholder](https://miro.medium.com/max/573/1*a-qmcT5oChQUWx2JlXWNLw.png "Large example image")

We now try this on both Book Title and Author section and then see the results

![placeholder](https://miro.medium.com/max/573/1*ZZX2komn6crnT9oPVi6OKA.png "Large example image")

We got the contents of ***/etc/passwd*** file, we see that there is a user named ***reader***, so we will try to get the ssh keys of it using the below command

```<script>x=new XMLHttpRequest;x.onload=function(){document.write(this.responseText)};x.open("GET","file:///home/reader/.ssh/id_rsa");x.send();</script>```

![placeholder](https://miro.medium.com/max/573/1*fe0wpv4J5F4XU1TnqpRK9w.png "Large example image")

We got the ssh key for the user read, but when try to copy the key from here , we were having some issues as it wasnt copying in properly format

So we use ghostscript to copy the contents of the pdf into a file

![placeholder](https://miro.medium.com/max/573/1*ywX1gBw084nhDJGISiM6Mw.png "Large example image")

We see we copied the key file along with some unwanted text which we later remove it manually

![placeholder](https://miro.medium.com/max/573/1*x99cTzdBtIiNRA-T1gK6SQ.png "Large example image")

We connected to user ***reader*** and now moving on to get user flag

![placeholder](https://miro.medium.com/max/247/1*8Lc_vb8ob9BLcc5defoWng.png "Large example image")

# Privilege Escalation

Running ***PSPY*** , we see that logrotate is being ran by root during few intervals

![placeholder](https://miro.medium.com/max/573/1*63R_9n0U46ojZP0YMH2hdA.png "Large example image")

We found an exploit for it and before that have to prepare our payload

![placeholder](https://miro.medium.com/max/573/1*MiP35qXB5ptxteFhJk5cLA.png "Large example image")

Now we run the **exploit** and see the **backups** directly which was in home folder of user ***reader*** which contained log files for logrotate

![placeholder](https://miro.medium.com/max/457/1*yeW56cDR-Eg8kpNKo2Srww.png "Large example image")

We copied so that it triggers the exploit and then we see

![placeholder](https://miro.medium.com/max/573/1*KXs0qJT6OJXMQF-ijLeNEg.png "Large example image")

We see that it copied the root ssh key to our created directory and we can read it below

![placeholder](https://miro.medium.com/max/573/1*a8nAgOz3hmI19o671jiEjw.png "Large example image")

Now we move into connecting to root user

![placeholder](https://miro.medium.com/max/573/1*6sxHAv4GR4185pDbKjrZ6g.png "Large example image")

Moving onto getting the root flag

![placeholder](https://miro.medium.com/max/573/1*79mb9GcNdt6QMuvg_Pv1uQ.png "Large example image")

# References

<a href="https://tech.feedyourhead.at/content/abusing-a-race-condition-in-logrotate-to-elevate-privileges">Abusing a race condition in logrotate to elevate privileges</a>

<a href="https://resources.infosecinstitute.com/sql-truncation-attack/"> SQL Truncation Attack</a>

<a href="https://www.noob.ninja/2017/11/local-file-read-via-xss-in-dynamically.html?m=1">Local File Read via XSS in Dynamically Generated PDF </a>
