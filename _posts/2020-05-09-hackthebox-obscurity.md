---
date: 2020-05-09 12:26:40
layout: post
title: HackTheBox - Obscurity
subtitle: HackTheBox Writeup - Obscurity
description: Obscurity is a box which is completely based on python codes where we exploit them one by one to get multiple users and also the root.
image: https://cdn-images-1.medium.com/max/1600/0*UJ7LCAnvHyqJrswx
optimized_image: https://cdn-images-1.medium.com/max/1600/0*UJ7LCAnvHyqJrswx
category: HackTheBox
tags:
  - hackthebox
  - python
  - hacking
  - penetration testing
  - ctf
  - infosec
author: ferllen
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a> . Also join me on discord.
The IP of this box is 10.10.10.168

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/1600/1*XPRXOljrrQSQZQKBVO-51g.png "Large example image")

We get 2 Open Ports and 2 Closed Ports, so we now run service scan for each one of those open ports

![placeholder](https://cdn-images-1.medium.com/max/2400/1*XdFF3IfDDUEB8xo5cDOxJw.png "Large example image")

So on the Open Ports we see that we have one for SSH and other for Web Part on 22 and 8080 respectively. I still don't know why NMAP gave results for Closed Ports
Moving onto the web part

## Port 8080 - Web

Loading the website in the browser with the port

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see that its a webpage related to security , scrolling further more

![placeholder](https://miro.medium.com/max/1400/1*i17IzkmR36PwlYovJiXU1Q.png "Large example image")


We see we have a Development section and also we a message to the server devs that the source code of the web server running is in a secret development directory by the filename <em><strong>"SuperSecureServer.py"</strong></em>

![placeholder](https://miro.medium.com/max/2000/1*qxqRJBrIJ3WHTGsfCt9uRQ.png "Large example image")

So we run a simple bash script to automate the bruteforcing of directory search and then we get a successful hit , i.e, develop
Now trying to access the source code

![placeholder](https://miro.medium.com/max/1400/1*5JTES0r5IvohkHvU29foiA.png "Large example image")

![placeholder](https://miro.medium.com/max/1380/1*LIJ_71e3cjGPcRii4hYowg.png "Large example image")

![placeholder](https://miro.medium.com/max/1238/1*6kIGbyDs_kJoFxUxEYVQ4w.png "Large example image")

We see its a long code, but reading carefully each functions, we get one which looks interesting

![placeholder](https://miro.medium.com/max/1186/1*nLrV-BIeduTn7MbW9V8btw.png "Large example image")

On the function ***serveDoc***, we see that it tries to use the exec function on the path, so I just copy some lines of the function to my local machine and try to exploit it manually first

![placeholder](https://miro.medium.com/max/788/1*kamCgRQbJ0Pg25BMaCMJgA.png "Large example image")

These lines were copied and now we execute our python script

![placeholder](https://miro.medium.com/max/918/1*f6nrVKD6sM7RACkuhIu3Tw.png "Large example image")

We see that we can successfully perform code execution by just escaping with a semicolon followed by the ***os.system*** function which will then be followed by our system commands

![placeholder](https://miro.medium.com/max/1400/1*fBS24p-EuT_z4gf4Qj5hKQ.png "Large example image")

So here we try to get reverse shell and looking back to the netcat listener

![placeholder](https://miro.medium.com/max/1050/1*FAf0Of8bKxXuNPjr8FRalQ.png "Large example image")

We got reverse shell successfully, looking further to what we have here

![placeholder](https://miro.medium.com/max/1400/1*Y6myNqUNfd3i-6m5_4YY2A.png "Large example image")

We have a user folder ***robert*** and inside that folder we see alot of **txt** files and also **py** files , we also have the *user.txt* flag which we currently cant read as only user ***Robert*** has the permissions to read it
We also see an interesting python script ***SuperSecureCrypt.py*** , looking at the code

![placeholder](https://miro.medium.com/max/1400/1*MZNpV_aXQjoO0hnvomgASw.png "Large example image")

Looking more into the code

![placeholder](https://miro.medium.com/max/1400/1*0hbJBdxMe4se3jLjIRRsYQ.png "Large example image")

The code uses addition and modulo to encrypt/decrypt the files and we see that we have two text files **check.txt** and **passwordreminder.txt**

![placeholder](https://miro.medium.com/max/1400/1*Gzi9TwoOX9_xawNEOppRHA.png "Large example image")

We see one text file has a clear text message and the other has the encrypted, so there might be some kind of XOR happening
Here now I use a python code to reverse the encryption

![placeholder](https://miro.medium.com/max/1400/1*Za2CrGUeCo_Dno4T0leMsg.png "Large example image")

Now we run the script

![placeholder](https://miro.medium.com/max/1400/1*mwi4D4a8W3OQZGPrGHY_gA.png "Large example image")

We see we got a name **alexandrov** , which actually wasnt fully decrypted and I needed a help from a friend here. So the full decryption was **alexandrovich**

![placeholder](https://miro.medium.com/max/1400/1*G_3-15SGfbTNukCuhju-vw.png "Large example image")

We now move onto decrypting the **passwordreminder.txt** using the key we got and save it to a txt file in the tmp folder and after reading the file , we see the password is ***SecThruObsFTW***

![placeholder](https://miro.medium.com/max/2000/1*sDYJ8qpMkl__k9ATGX6E4Q.png "Large example image")

We tried connecting to user ***robert*** through SSH with the password which we decrypted and successfully logged in and got the user flag access

## Privilege Escalation

Running the ***sudo -l*** command

![placeholder](https://miro.medium.com/max/2000/1*n7mLrbaONpw9I3j_s4XGvw.png "Large example image")

We see that we can run sudo as user robert without password on ***/usr/bin/python3 /home/robert/BetterSSH/BetterSSH.py***
Checking the ***BetterSSH.py*** code

![placeholder](https://miro.medium.com/max/1400/1*rkgZO16Z2eI5EK9_r-HrjQ.png "Large example image")

Looking further more down

![placeholder](https://miro.medium.com/max/1400/1*nAnfz0YQwnV5aOlo4dZqqA.png "Large example image")

From the above code, we can see that if we have an authenticated session, it runs ***sudo -u `user`***

![placeholder](https://miro.medium.com/max/1400/1*A6HH6biwW0CYC40BMf7VNA.png "Large example image")

We got authenticated and then return back to a shell, using the ***-l*** , we get an Error in the output and then we see that we get the usage of the sudo command

![placeholder](https://miro.medium.com/max/1400/1*vZtpWGrSLaoetG4kcJBJ4w.png "Large example image")

So now we know its running *sudo* already , we just append down the below commands

![placeholder](https://miro.medium.com/max/1068/1*Ev6tV3l0-mNbZsxhhd845Q.png "Large example image")

This commands leads us to run bash as **root** user from *sudo* command, but we don't get any output back, hence, we try to get reverse shell and checking back to the netcat listener

![placeholder](https://miro.medium.com/max/1028/1*qRDyb8kIyZInMsAjrKEw2Q.png "Large example image")

We got reverse shell as root successfully, moving back to get the root flag

![placeholder](https://miro.medium.com/max/794/1*ZsDuJcjCOW_jTkWBamQYOQ.png "Large example image")










