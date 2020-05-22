---
date: 2020-04-11 12:26:40
layout: post
title: HackTheBox - Traverxec
description: Traverxec is an easy box that start with a custom vulnerable webserver with an unauthenticated RCE that we exploit to land an initial shell. After pivoting to another user by finding his SSH private key and cracking it, we get root through the less pager invoked by journalctl running as root through sudo.
image: https://miro.medium.com/max/573/1*amuZLJcqEZD5PCIRSc-bFg.png
optimized_image: https://miro.medium.com/max/573/1*amuZLJcqEZD5PCIRSc-bFg.png
category: HackTheBox
tags:
  - ctf
  - hackthebox
  - infosec
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a> . Also join me on discord.

The IP of this box is **10.10.10.165**

Port Scan

Running masscan on it , we get

`masscan -p1-65535,U:1-65535 10.10.10.165 --rate=200 -e tun0`

![placeholder](https://miro.medium.com/max/573/1*AJT6twCUtWs9rznnIWSlLw.png "Large example image")

Running NMAP against the open ports

![placeholder](https://miro.medium.com/max/573/1*gh2Yv_ntRL5k_dpyRJwlvg.png "Large example image")

Port 22 running **OpenSSH 7.9p1** and Port 80 running **nostromo 1.9.6**

Moving to the Web Part

## Port 80 — nostromo 1.9.6

Running the IP on the browser

![placeholder](https://miro.medium.com/max/810/1*6_svhw3D54PE_js-uqKgTQ.png "Large example image")

We see a webpage related to web development, but since we saw the web server was **nostromo 1.9.6** I went to search for potential exploits for it

![placeholder](https://miro.medium.com/max/810/1*ER7KTx-RACIcNixHR_dWLg.png "Large example image")

We see 2 exploit, one of them being on metasploit so we will use that one for our ease

Spawning metasploit and searching the module

![placeholder](https://miro.medium.com/max/810/1*0rzU1yEJW8BRtTFDlWzfFA.png "Large example image")

Checking and setting the required options

![placeholder](https://miro.medium.com/max/810/1*FVLlSvRSRRc6lS6KBmyrEw.png "Large example image")

Now we run the ***exploit*** command

![placeholder](https://miro.medium.com/max/810/1*ortiNnHThtHxa_I8ts1iXQ.png "Large example image")

We got shell on the box as www-data, but since this is an unstable shell , so I wish go for little bit more stable shell by using bash oneliner reverse shell command

![placeholder](https://miro.medium.com/max/1808/1*IZ85eeLrk67JmRFsUJJ34A.png "Large example image")

Looking back to the netcat listener

![placeholder](https://miro.medium.com/max/1280/1*yU5TUje4acXjoSNJSWrnfg.png "Large example image")

Looking further inside we get a folder for ***nostromo***

![placeholder](https://miro.medium.com/max/1104/1*U2vdfb6kxehhwqVhBiaTSw.png "Large example image")

Here we went into the **conf** folder of nostromo and then upon looking on the **nhttpd.conf** file we see that there is a **.htpasswd** file on the conf folder

![placeholder](https://miro.medium.com/max/1102/1*OKvEeyRvLy8m3U1soNFvZA.png "Large example image")

We got a hash along with the username ***david*** and looking at the hash type in hashcat website

![placeholder](https://miro.medium.com/max/1078/1*SdZkL_7OYGSoz3QFlU30wA.png "Large example image")

We see that its md5crypt or MD5(Unix) , since we now the mode to use which is 500, I will use hashcat for cracking it

![placeholder](https://miro.medium.com/max/1808/1*WlYX9Xx8j9V-1Br8CAmRHQ.png "Large example image")

We cracked the password which is ***Nowonly4me***, trying to login to the user david

![placeholder](https://miro.medium.com/max/962/1*iJgTGiaoEgc0A1tMCXIe9Q.png "Large example image")

We see that it fails, remember from the above config file we saw there was a **public_www** which was listed as **homedir_public**

If you go to the home directory of user ***david***

![placeholder](https://miro.medium.com/max/1004/1*N0yTkOgfiMA6RJYDVd9fMQ.png "Large example image")

We see that we cant see the contents of the folder, but if we try to access the **public_www** folder inside it

![placeholder](https://miro.medium.com/max/1266/1*liVW2L02Xyn7PqYqy3RerA.png "Large example image")

We got into that folder and also there is one more folder named **protected-file-area** so we try to access that

![placeholder](https://miro.medium.com/max/1280/1*YvNv_OJgdIOkpmeNXHN4jw.png "Large example image")

We see two files named **.htaccess** and **backup-ssh-identity-files.tgz**

![placeholder](https://miro.medium.com/max/1280/1*L7VMKasbIlt4U_HZbHx4mQ.png "Large example image")

We got a message in .htaccess file , looking into the other file type

![placeholder](https://miro.medium.com/max/1808/1*octOiVYQiJusna6E9Zq-ng.png "Large example image")

We see its a gzip compressed file, so we will bring it to our box

![placeholder](https://miro.medium.com/max/1280/1*jVcPH82qwL8zDHxDKAhdvg.png "Large example image")

![placeholder](https://miro.medium.com/max/1280/1*a5hxkj7B74BpGjROVBSOag.png "Large example image")

So here we used the base64 encoding method to bring the file to our box and now we work on it

![placeholder](https://miro.medium.com/max/1280/1*a5u1B45am24S-r9awAqQ5A.png "Large example image")

Here we decompressed the gzip tar file and we see that we got ssh keys

![placeholder](https://miro.medium.com/max/1280/1*a5Std2wsqzNH_vemeKavZg.png "Large example image")

We see that the ssh key is encrypted, so we will have to crack the passphrase

![placeholder](https://miro.medium.com/max/1808/1*pYVdmmPlGo_V7sHY7gwUfw.png "Large example image")

Now we crack it using **John**

![placeholder](https://miro.medium.com/max/1280/1*V3FjJDS1C5CxOVNh43ZvRw.png "Large example image")

We cracked it successfully and got the passphrase as ***hunter***

![placeholder](https://miro.medium.com/max/1280/1*cbubJZpX0pER63V_zYnchA.png "Large example image")

We got in successfully and now time to get the user flag

![placeholder](https://miro.medium.com/max/676/1*pNvo3BNt-5iXr6bgXsr8dw.png "Large example image")

Moving onto the priv esc part

## Privilege Escalation

Looking into the directory of the user david

![placeholder](https://miro.medium.com/max/1242/1*1LWppGKB00W-Job2FOkfgA.png "Large example image")

We see a folder named bin in the home directory of user david and also inside that folder are two files one of them being a bash script

![placeholder](https://miro.medium.com/max/1280/1*vNPpfSAATIYaAueMIO2ueg.png "Large example image")

We see that last time which uses sudo, but when we use sudo

![placeholder](https://miro.medium.com/max/754/1*1NqMwjueyKbzTNzZ2Ttg8Q.png "Large example image")

It asks for password which we dont have, but when we use the last line command

![placeholder](https://miro.medium.com/max/1808/1*LKSL0VyR2WjpRmdP1xeP_w.png "Large example image")

Upon looking on GTFOBins, I figured it out we could exploit the journalctl binary if it gets us a prompt but for that we need to change out terminal to tty

![placeholder](https://miro.medium.com/max/1280/1*Baxf6WV9VMOrKFsRvsij9A.png "Large example image")

We see we get a prompt at the end of the line, so now we abuse it to get the a shell

![placeholder](https://miro.medium.com/max/1280/1*UgjRchu97OCeeBlKOd8CUQ.png "Large example image")

After pressing enter

![placeholder](https://miro.medium.com/max/800/1*LaWPTeEz2pG0SMzw2fwNeA.png "Large example image")

We got root!!! Root flag in the usual place as always

![placeholder](https://miro.medium.com/max/223/1*aNO6ezz1W0SAAQAkOWqGOQ.png "Large example image")

Overall a big troll at the end :)

# Youtube Video

<iframe width="420" height="315" src="https://www.youtube.com/embed/fLlD9XemQfU?autoplay=1"></iframe> 
 

# References

<a href="https://github.com/rapid7/metasploit-framework/pull/12476">Nostromo Exploit</a>

<a href="https://gtfobins.github.io/gtfobins/journalctl/">journalctl | GTFOBins</a>

