---
date: 2020-02-05 12:26:40
layout: post
title: HackTheBox - OpenAdmin 
subtitle: HackTheBox Walkthrough - OpenAdmin
description: OpenAdmin was an easy box which had a vulnerability on one of its service OpenNetAdmin and then privesc using sudo
image: https://miro.medium.com/max/573/0*0-3G2TuC64cvN3DN
optimized_image: https://miro.medium.com/max/573/0*0-3G2TuC64cvN3DN
category: HackTheBox
tags:
  - hackthebox
  - ctf
  - infosec
  - hacking
author: ferllen
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a>. Also join me on discord.

The IP of this box is **10.10.10.171**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/573/1*xlLMwHoLCJxrbmdLtmipKw.png "Large example image")

We get only 2 Open Ports running SSH and Apache Web Server

## Web Part

The main root directory in the web part gave us default Apache webpage so we move onto Gobuster the web

![placeholder](https://miro.medium.com/max/810/1*8LJev4qBb5YXyXqJw9VXkA.png "Large example image")

We got few entries, one of which was ***/music***, looking further into it

![placeholder](https://miro.medium.com/max/810/1*r_pMowYupKQpzFdt5dxjJQ.png "Large example image")

We see a webpage, clicking on the ***Login*** option

![placeholder](https://miro.medium.com/max/810/1*ar8h3F2eyMRv0M24ERVNKQ.png "Large example image")

We get redirected to ***/ona/*** page, we can see that it is running **OpenNetAdmin v18.1.1**

Searching for exploits on searchsploit

![placeholder](https://miro.medium.com/max/810/1*QKm2Dh-wWfY8tcRwGOkaoQ.png "Large example image")

We get 2 exploitings for the existing version, we use the script instead of metasploit

Getting the script to our box and then checking the contents of it

![placeholder](https://miro.medium.com/max/810/1*3WwbSkTwlYd6XvpZTgwr4w.png "Large example image")

We just use the curl command in it, but we change the $CMD to the command of our desire

![placeholder](https://miro.medium.com/max/810/1*t1aW1NUStSSQG7kpxKLtyA.png "Large example image")

Here we use the curl command to ping to our local machine

![placeholder](https://miro.medium.com/max/573/1*fWSwyftgmApItXaWr1bAgA.png "Large example image")

As we can see that we got pinged back successfully

![placeholder](https://miro.medium.com/max/810/1*NO0JZ5e-_JASJ4rb5tmHdQ.png "Large example image")

Now we upload a PHP reverse shell through the above command

![placeholder](https://miro.medium.com/max/494/1*igekOgB0YMqB9BPDMSVTEA.png "Large example image")

Now we access the reverse shell php script and then check the netcat listener

![placeholder](https://miro.medium.com/max/573/1*C7pNRcD8R5kYM10hqm5l-Q.png "Large example image")

We got reverse shell as ***www-data*** user, upon enumerating more

![placeholder](https://miro.medium.com/max/573/1*lROPiIJ2kzP5bB1E7qYKNg.png "Large example image")

We see a database_settings php script on the ***/opt/ona/www/local/config*** folder and inside of that script , we get a password so we use it for user jimmy through SSH

![placeholder](https://miro.medium.com/max/573/1*hONzi3Tx2iX0Ij4FGfpSDg.png "Large example image")

We connected to jimmy user through SSH successfully

![placeholder](https://miro.medium.com/max/573/1*ludrwLn6Uz08lnff8g6Tpg.png "Large example image")

Running the id command , we see that we are groups with ***1002(internal)***

Checking for files with group permission ***internal***

![placeholder](https://miro.medium.com/max/443/1*lZ6sP3QGL8h18a4LVUY3Ww.png "Large example image")

We get ***/var/www/html*** folder and some php files with the group permission of it

![placeholder](https://miro.medium.com/max/573/1*tfQmzvqCSuVg1HLGgh32hw.png "Large example image")

Checking the contents of **main.php** we see that it runs ***shell_exec*** function which will **cat the id_rsa file of joanna**

But we didnt see any web stuff running other than what we got before, so checking for open ports listening locally

![placeholder](https://miro.medium.com/max/573/1*1DQRRBxHc56sr8tH-S4OPg.png "Large example image")

We see that port **52846** is open and listening, so we port forward it to our local machine through SSH

![placeholder](https://miro.medium.com/max/573/1*eYhRgPucu9JpkJI8ehTEuQ.png "Large example image")

We port forwarded to our local machine, now we check the port in the web browser

![placeholder](https://miro.medium.com/max/573/1*-avZ8-mEOZzbPOCr1zkKXg.png "Large example image")

We get redirected to login page login.php, as we remember that there was a php file in **/var/www/internal/login.php**, checking the contents of the file

![placeholder](https://miro.medium.com/max/810/1*0KzNOygXv9pFy1ea8EXXgw.png "Large example image")

We see that we have a username and password, but the password is *SHA512* encrypted so we use an online decrypter for it

![placeholder](https://miro.medium.com/max/810/1*hb8HyYBuqyF_NiqQHdlTUA.png "Large example image")

We decrypted the password,i.e, ***Revealed***

We now login and then get redirected to **main.php** which reveals SSH Key

![placeholder](https://miro.medium.com/max/460/1*Z0ci0QhawsWSXaggUdeFGg.png "Large example image")

We see that it is encrypted SSH keys, we run **sshng2john** and saved it to a file

![placeholder](https://miro.medium.com/max/810/1*Cs9m-c80VHWYSdbuFODQ_g.png "Large example image")

We now use **john** to crack the passphrase

![placeholder](https://miro.medium.com/max/573/1*SBFeI2K-pImVXAmlPvMTFA.png "Large example image")

We cracked the passphrase successfully,i.e, ***bloodninjas***

Now we connect to user ***joanna*** through SSH

![placeholder](https://miro.medium.com/max/573/1*ec4SfKvz8rqDNQ-_CugmTQ.png "Large example image")

Connection successfully through SSH with user joanna and now move onto getting the user flag

![placeholder](https://miro.medium.com/max/573/1*7mL63h1fhO7m00X639ms5w.png "Large example image")

Now time for privilege escalation

## Privilege Escalation

Running ***sudo -l*** command

![placeholder](https://miro.medium.com/max/573/1*4R1kMYSBwN23hW001Tm-GQ.png "Large example image")

We see that user joanna can run ***/bin/nano on /opt/priv*** with sudo without password

We run the command and then move further to get shell as root

We press **CTRL+R and then CTRL+X**

![placeholder](https://miro.medium.com/max/573/1*Aj9X3Zv0zMMM9-rdfCnQ1A.png "Large example image")

We can see we get prompted to put command to execute

We run ***reset; sh 1>&0 2>&0*** and then press Enter

![placeholder](https://miro.medium.com/max/810/1*HPVRTaYXqyr_G3vY4hqywg.png "Large example image")

We got shell as root, which we can confirm down below too

![placeholder](https://miro.medium.com/max/116/1*2DKGw9c7zYMG9wezesnlfA.png "Large example image")

Now moving onto getting root flag

![placeholder](https://miro.medium.com/max/573/1*TZGxlVFjieHyWZerImJiPA.png "Large example image")

# References

<a href="https://gtfobins.github.io/gtfobins/nano/#sudo">Nano Sudo Privilege Escalation</a>








