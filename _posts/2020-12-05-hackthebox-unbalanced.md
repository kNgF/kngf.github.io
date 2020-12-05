---
date: 2020-12-05 12:26:40
layout: post
title: HackTheBox - Unbalanced
description: Hard Linux Machine containing rsync, squid proxy, cachemgr, pi-hole in the learning
image: https://miro.medium.com/max/2400/0*HUTMestJEieq6B-W
optimized_image: https://miro.medium.com/max/2400/0*HUTMestJEieq6B-W
category: HackTheBox
tags:
  - linux
  - hard
author: feodore
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">feodore</a>. Also join me on discord.

The IP of this box is **10.10.10.200**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/700/0*ZnmlExcqJsvKcKvC.png "Large example image")

We see 3 Open Ports , Port **22** for **SSH** , Port **873** for **Rsync** and Port **3128** for **Squid**

## Way To User

Since we have Rsync on the box, we just enumerate it and get the directories available

![placeholder](https://miro.medium.com/max/563/1*OV7xXF6GEPODNTw-VHRvpA.png "Large example image")

We have a folder **conf_backups** and we try to see the contents of that folder

![placeholder](https://miro.medium.com/max/700/1*AmVI084mtEYMNDHymEQKWg.png "Large example image")

We downloaded the files to our local machine and see that the files are encypted but we dont know currently what encryption is being used

![placeholder](https://miro.medium.com/max/1000/1*wF4uSZv_WyMHgIQCH5zdjQ.png "Large example image")

Checking the **.encfs6.xml** file shows us that the encryption being used here is **EncFS 1.9.5**

![placeholder](https://miro.medium.com/max/700/1*DR8PcgS7k5itrwBYNC5yfA.png "Large example image")

Since this encryption is password based, we need to crack the password in order to decrypt the file, there is a python script which converts the encryption to the format crackable with John and cracked the password , i.e, ***bubblegum***

![placeholder](https://miro.medium.com/max/1000/1*QObRcdd7SyTTfqcv-2RfdQ.png "Large example image")

Now we decrypt the files and see that there are alot of config files, and the one interesting here is the ***squid.conf*** file since we have Squid installed on the machine

![placeholder](https://miro.medium.com/max/1000/1*c2zfQDZl8fJ1xOFzgye3Jw.png "Large example image")

Checking the enabled configurations on this file, we see that we have a domain ***intranet.unbalanced.htb*** which we put on our hosts file and also we see ***cachemgr_passwd*** being used where we have the password as well as the cache object being used

![placeholder](https://miro.medium.com/max/1000/1*lX0VHVRS2wr2FwSxMciEPg.png "Large example image")

Checking the domain through the Squid proxy settings on the web, we see a nice web page but nothing much here

![placeholder](https://miro.medium.com/max/1000/1*GQFTNJEwt772ymWbp2gGiw.png "Large example image")

Getting some more details from the Squid leads us to 2 more extra domains along with their IP

![placeholder](https://miro.medium.com/max/700/1*GjF-L-qk7yMRLCvMi-yxHQ.png "Large example image")

Checking both of the domains through the IP, it is same as the first domain which we got before where we couldn’t do anything as nothing was dynamic here

![placeholder](https://miro.medium.com/max/1000/1*-fZdIJxQROk0YXtRpZUlDg.png "Large example image")

The ***intranet-host2.unbalanced.htb*** being the same as well

![placeholder](https://miro.medium.com/max/1000/1*HcF9x0JGMF7Zc_xCfCWrPw.png "Large example image")

We didnt get anything interesting , but looking at the domains and the IP , we can conclude that there might be a ***intranet-host1.unbalanced.htb*** with the IP **172.31.179.1** and checking that we see it has been taken out due to security issues

![placeholder](https://miro.medium.com/max/700/1*QhrAhjAXOEZ4n-JQ6ujlUQ.png "Large example image")

Since every domain were redirecting to the intranet.php page, we can assume that this domain should be having the same page and indeed it had the same page but here the Employee login functionality was working and we tried multiple injections

![placeholder](https://miro.medium.com/max/1000/1*PGObqAetKm9M0LixTNXYcQ.png "Large example image")

After trying XPATH Injection , we got some results

![placeholder](https://miro.medium.com/max/608/1*pvGqh41nxClaLmqkTNiLbg.png "Large example image")

We found some users, but we don’t have the password so we try to get the password but before that we need to be sure about the password length of each user which we do here by doing a word count on the request and the correct one gives the unique answer

![placeholder](https://miro.medium.com/max/1000/1*0FIXjQQCjDDjHtbsshKHCQ.png "Large example image")

So we automate the stuff using python scripting after knowing the password length to get the password

![placeholder](https://miro.medium.com/max/700/1*J3fwV9tVcKpIrPkwwMtAxQ.png "Large example image")

Connecting through the SSH with the leaked credentials, we get in as user ***bryan*** which has the user flag

![placeholder](https://miro.medium.com/max/700/1*x6ajJEB00WF4d5R9Ixw6Rg.png "Large example image")


## Way To Root

On the home directory of the user ***bryan***, there is **TODO** text file which tells us that **Pi-Hole** has been installed through ***docker*** which is listening locally, upon searching we know its listening on port **8080**

![placeholder](https://miro.medium.com/max/700/1*Ay8oV-tkvVZs3XT-NlebeQ.png "Large example image")

There was a recently ***CVE*** for ***Pi-Hole*** whose exploit we download on our local machine and then run and get shell as ***www-data*** on docker container of Pi-Hole

![placeholder](https://miro.medium.com/max/700/1*Tsjk6K9F8rN6KnbStGknog.png "Large example image")

The /root directory is readable to all and we see the pihole_config.sh file here on which we have a password

![placeholder](https://miro.medium.com/max/700/1*7vztiAfCN-LNzE_0te_3qA.png "Large example image")

Using that password on root lets us in and we get root on the main machine and then can get the root flag

![placeholder](https://miro.medium.com/max/400/1*YaPkg5VvVTHsYhrDzeAPLg.png "Large example image")


# References

<a href="https://github.com/AndreyRainchik/CVE-2020-8816/">CVE-2020-8816 Github POC</a>

<a href="https://wiki.squid-cache.org/Features/CacheManager">CacheMGR Wiki</a>

<a href="https://medium.com/@minimalist.ascent/enumerating-rsync-servers-with-examples-cc3718e8e2c0">RSync Enumeration</a>




<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










