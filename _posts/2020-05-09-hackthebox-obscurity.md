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
author: ferllen
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

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")


We see we have a Development section and also we a message to the server devs that the source code of the web server running is in a secret development directory by the filename <em><strong>"SuperSecureServer.py"</strong></em>

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

So we run a simple bash script to automate the bruteforcing of directory search and then we get a successful hit , i.e, develop
Now trying to access the source code

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see its a long code, but reading carefully each functions, we get one which looks interesting

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

On the function ***serveDoc***, we see that it tries to use the exec function on the path, so I just copy some lines of the function to my local machine and try to exploit it manually first

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

These lines were copied and now we execute our python script

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see that we can successfully perform code execution by just escaping with a semicolon followed by the ***os.system*** function which will then be followed by our system commands

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

So here we try to get reverse shell and looking back to the netcat listener

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We got reverse shell successfully, looking further to what we have here

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We have a user folder ***robert*** and inside that folder we see alot of **txt** files and also **py** files , we also have the *user.txt* flag which we currently cant read as only user ***Robert*** has the permissions to read it
We also see an interesting python script ***SuperSecureCrypt.py*** , looking at the code

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

Looking more into the code

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

The code uses addition and modulo to encrypt/decrypt the files and we see that we have two text files **check.txt** and **passwordreminder.txt**

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see one text file has a clear text message and the other has the encrypted, so there might be some kind of XOR happening
Here now I use a python code to reverse the encryption

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

Now we run the script

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see we got a name **alexandrov** , which actually wasnt fully decrypted and I needed a help from a friend here. So the full decryption was **alexandrovich**

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We now move onto decrypting the **passwordreminder.txt** using the key we got and save it to a txt file in the tmp folder and after reading the file , we see the password is ***SecThruObsFTW***

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We tried connecting to user ***robert*** through SSH with the password which we decrypted and successfully logged in and got the user flag access

## Privilege Escalation

Running the ***sudo -l*** command

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see that we can run sudo as user robert without password on ***/usr/bin/python3 /home/robert/BetterSSH/BetterSSH.py***
Checking the ***BetterSSH.py*** code

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

Looking further more down

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

From the above code, we can see that if we have an authenticated session, it runs ***sudo -u `user`***

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We got authenticated and then return back to a shell, using the ***-l*** , we get an Error in the output and then we see that we get the usage of the sudo command

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

So now we know its running *sudo* already , we just append down the below commands

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

This commands leads us to run bash as **root** user from *sudo* command, but we don't get any output back, hence, we try to get reverse shell and checking back to the netcat listener

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We got reverse shell as root successfully, moving back to get the root flag

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")










