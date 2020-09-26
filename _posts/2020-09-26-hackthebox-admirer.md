---
date: 2020-05-02 12:26:40
layout: post
title: HackTheBox - Admirer 
subtitle: HackTheBox Walkthrough - Admirer
description: Admirer was a medium level machine which had vulnerability of web for Admirer and privilege escalation through python library hijacking
image: https://miro.medium.com/max/700/0*fndFOmHDZqzrYyhV
optimized_image: https://miro.medium.com/max/700/0*fndFOmHDZqzrYyhV
category: HackTheBox
tags:
  - hackthebox
  - linux
author: anishka
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">anishka</a>. Also join me on discord.

The IP of this box is **10.10.10.187**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/678/1*cqq3cZVOZQ6FS5Kvb2cCSQ.png "Large example image")

We have only 3 Open Ports for FTP , SSH and Web respectively

Moving to the Web Part

## Port 80 — Web

Running the IP in the browser , we see a wonderful web page

![placeholder](https://miro.medium.com/max/1000/1*XyqkW6yRwstEnxAr8duIHg.png "Large example image")

Checking the ***robots.txt*** file, we get

![placeholder](https://miro.medium.com/max/700/1*i33_cnu_JDls522V0kIefQ.png "Large example image")

We get a disallow on ***/admin-dir*** , which was unaccessible to us as no permissions, I ran gobuster against that directory and got few results

![placeholder](https://miro.medium.com/max/237/1*iatRyIOcNwLmn_9XnD8iDQ.png "Large example image")

On the **contacts.txt** file, we had something but not that interesting

![placeholder](https://miro.medium.com/max/527/1*A2gI4s9zRBdKVui0-EkenA.png "Large example image")

On the other side, on credentials.txt we have some credentials , one of which is for FTP

![placeholder](https://miro.medium.com/max/540/1*oJ8TqoC5Re3cL97F7Cjerg.png "Large example image")

So we log into FTP from the creds we got

![placeholder](https://miro.medium.com/max/296/1*kX7y9K78LELLtuAuG6LFMg.png "Large example image")

Now we check the contents and get all the files inside the ftp to our local machine

![placeholder](https://miro.medium.com/max/522/1*JMOKfORzDvgtoA_WkcBoxQ.png "Large example image")

One of the files contains the backup of the web in the ***html.tar.gz*** file

![placeholder](https://miro.medium.com/max/700/1*Tvohh6F002NJXaKJW5gdhw.png "Large example image")

After unzipping the file , we see it has a web directory named ***utility-scripts*** which couldn’t get from gobuster on main directory, inside of that folder there are 4 PHP files

![placeholder](https://miro.medium.com/max/552/1*Yk71ktC5EOnJFCwZRupFLw.png "Large example image")

On the ***db_admin.php***, We see some database credentials, but we dont find any ways of using these credentials so I ran gobuster against the ***utility-scripts*** directory

![placeholder](https://miro.medium.com/max/220/1*UgnTK85nrr454BoXgpJklw.png "Large example image")

We found an extra PHP file named ***adminer.php***

Checking on the web

![placeholder](https://miro.medium.com/max/668/1*mouqX26rvsJ-At_ye5TdhQ.png "Large example image")

We have **Adminer 4.6.2** Login Page, but the creds we got before didn't worked as it didn't had the DB name

Also the backup file had the backup on ***index.php*** file , which contained some creds too

![placeholder](https://miro.medium.com/max/542/1*qStg-oSO5rjldTy2Gww9sg.png "Large example image")

Here we have different credentials with **dbname** too , trying these on the login page

![placeholder](https://miro.medium.com/max/386/1*HVAhk2w9EyieOP8YwBmFQQ.png "Large example image")

Now we click on Login

![placeholder](https://miro.medium.com/max/521/1*RTQA1kdN9C5P9HeCJsr8cA.png "Large example image")

We get **Access Denied**, probably because the credentials are incorrect, so I will host my own server and then connect to it remotely from this **Adminer Login** page

![placeholder](https://miro.medium.com/max/413/1*83sMlCrfF0Fz1HRwxRT-wA.png "Large example image")

We are set to go and enter our DB

![placeholder](https://miro.medium.com/max/700/1*v_1gy6PVqeDASFTCH4iTpQ.png "Large example image")

We got logged in to our local DB server successfully and now we move onto the **SQL Command** section

![placeholder](https://miro.medium.com/max/658/1*PHGKHJm8eaJ0JlHAEsa_lA.png "Large example image")

We type the command in the above pic and then click on Execute , all this will do is insert the contents of ***index.php*** in the table **xml**

![placeholder](https://miro.medium.com/max/658/1*ozczHdt7RWaVrhCCDG7q_g.png "Large example image")

Now checking the contents of the table , we see that in the currently hosted index.php file, the credentials for the Admirer looks different from what we saw before , so we try the currently given creds

![placeholder](https://miro.medium.com/max/390/1*FrFEpAWix4qC8uoi_-Baag.png "Large example image")

Clicking on Login and we move towards the dashboard successfully

![placeholder](https://miro.medium.com/max/700/1*qPNBTXfENQXXxHTpq-nc9Q.png "Large example image")

Since we didn’t found anything useful here in the database , I tried the password with SSH and we got connected as Waldo successfully

![placeholder](https://miro.medium.com/max/541/1*8s46W3bxu9FdXyxsBckZvA.png "Large example image")

Now we get the user flag in the home folder of ***Waldo***

![placeholder](https://miro.medium.com/max/243/1*LsM9xWVZ2c6epPeq8lUuuw.png "Large example image")

## Privelege Escalation

Running the ***sudo -l*** command, we see

![placeholder](https://miro.medium.com/max/700/1*CbexAfGSXw7IInPPXhDAMA.png "Large example image")

We can SETENV with sudo on ***/opt/script/admin_tasks.sh*** , **SETENV** will allow us to provide an environment variable while using the sudo command

Looking on the bash script, we see that a function **backup_web** is calling a python script on ***/opt/script/backup.py***

![placeholder](https://miro.medium.com/max/586/1*pGBV9WHKrQmB1aMFjl2eGA.png "Large example image")

Checking the code of ***backup.py***, we see that it imports ***make_archive*** from ***shutil*** module

![placeholder](https://miro.medium.com/max/339/1*BfjApEHm0Zl-nXtNPuHoFQ.png "Large example image")

Since we don't have write permissions on the same directory as those files, and since we can **SETENV** with sudo as ***Waldo***, so we can just try **Python Module Path Hijacking**

![placeholder](https://miro.medium.com/max/485/1*B0-TEZdTAEnXBDI48F3syQ.png "Large example image")

So we create a simple python reverse shell

![placeholder](https://miro.medium.com/max/381/1*jFM428IfRmsNC9hE02sdnA.png "Large example image")

We now save this as ***make_archive.py*** and ***shutil.py*** and give all permissions

![placeholder](https://miro.medium.com/max/431/1*rNto_ArtxXlXJ0KcJaNbMg.png "Large example image")

Now we run our final command along with the netcat listener

![placeholder](https://miro.medium.com/max/1000/1*wSYuFYjZyu8lHS7bn8LbPQ.png "Large example image")

We got root and also the root flag


# References

<a href="https://rastating.github.io/privilege-escalation-via-python-library-hijacking/">Python Library Hijacking</a>

<a href="https://superuser.com/questions/363813/sudo-does-not-preserve-pythonpath">SETENV</a>

<a href="https://sansec.io/research/adminer-4.6.2-file-disclosure-vulnerability">Adminer-4.6.2 File Disclosure Vulnerability</a>

<a href="https://www.foregenix.com/blog/serious-vulnerability-discovered-in-adminer-tool">Adminer Vulnerability</a>


<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 









