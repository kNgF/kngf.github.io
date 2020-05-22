---
date: 2020-04-04 12:26:40
layout: post
title: HackTheBox - Registry
description: Registry is a hard box which has vulnerable Bolt CMS , Docker Registry and then restic service which leads to root.
image: https://miro.medium.com/max/573/0*HOujVpXcA-DPlziK
optimized_image: https://miro.medium.com/max/573/0*HOujVpXcA-DPlziK
category: HackTheBox
tags:
  - ctf
  - infosec
  - hacking
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a>. Also join me on discord.

The IP of this box is **10.10.10.159**

## Port Scan

Running nmap on it , we get

![placeholder](https://miro.medium.com/max/573/1*0Vkc7VUIGpkTeuaQ3mM5-w.png "Large example image")

We see 4 Open ports on port 22, 80 , 443, 8000 and 8001 respectively

Moving onto the Web Part

## Web Part

Checking the lowest web port, i.e, Port 80

![placeholder](https://miro.medium.com/max/810/1*hcaT2ZXj-S3dC8vlOau4qQ.png "Large example image")

We see an nginx server message, so we now run directory against this but this time I am gonna use **WFUZZ** instead of Gobuster

![placeholder](https://miro.medium.com/max/810/1*7o43UkX9sh_NoRhx5z43Gg.png "Large example image")

Well when fuzzing the main webpage, we got four entries out of which I first bruteforced the **/install** directory but end up with nothing interesting

![placeholder](https://miro.medium.com/max/810/1*ss3prNWp9-ugoF9fraQIhA.png "Large example image")

On fuzzing the **/bolt** directory, we got pretty few entries , so moving onto the **/bolt/bolt** directory

![placeholder](https://miro.medium.com/max/810/1*W0YXYeKOaoZRv4dUGQMyNg.png "Large example image")

We get redirected to a login page, so I bruteforced the login part for user **admin** using Burp Intruder

![placeholder](https://miro.medium.com/max/810/1*ip1fE_C50xzPoCfIRyA98w.png "Large example image")

We can see that the password ***strawberry*** gives us a **302** status code which means its redirecting us somewhere so we use it as our password

![placeholder](https://miro.medium.com/max/810/1*cnVFjm6hujP45lRkMrg7MQ.png "Large example image")

We got logged in successfully

Also we saw that we got an **install** directory before, looking further into it

![placeholder](https://miro.medium.com/max/810/1*8i94h1DEuWdDvdG9iXAMNw.png "Large example image")

We see some junk file, trying to get this onto our box using ***wget***

![placeholder](https://miro.medium.com/max/810/1*00Q6BbOeTUTyONVZ3SdDaA.png "Large example image")

We see that it is a zip file , so we extracted and see the contents

![placeholder](https://miro.medium.com/max/494/1*WXmjDWZAL5OVo7kH5NtqdA.png "Large example image")

We see two files, ***ca.crt*** and ***readme.md***, checking the readme.md file we see that it says that it has docker running so I will scan for vhosts and see that

![placeholder](https://miro.medium.com/max/810/1*LUkXlrtpffDaFe7a0Tkvzw.png "Large example image")

Wee see that we have ***docker.registry.htb***, so I have added the vhost into my **/etc/hosts** file

![placeholder](https://miro.medium.com/max/573/1*9ckebhMatFJqXsJWtzq2JQ.png "Large example image")

We see its nothing but blank, running WFuzz against for directory bruteforcing

![placeholder](https://miro.medium.com/max/810/1*hUfeLZPyZO_jFEJYntaWZw.png "Large example image")

We see that we have a directory named **v2**, looking further into it

![placeholder](https://miro.medium.com/max/810/1*QcG105CVkLaBlm0uM1an3g.png "Large example image")

We get a basic authentication, trying the default admin admin creds

![placeholder](https://miro.medium.com/max/573/1*uEmHPq60qasVIuvNjcPBWA.png "Large example image")

We get a blank API, checking the catalogs

![placeholder](https://miro.medium.com/max/573/1*3BJ4BS9okwA56WOWwJvKVw.png "Large example image")

Looking for the catalogs, we see that we have a repository named ***bolt-image***

![placeholder](https://miro.medium.com/max/573/1*t8vAGUucMkvWg7ORE6ay_g.png "Large example image")

Moving further on checking the tag lists, we see that we have a tag ***latest*** on the ***bolt-image*** repository

![placeholder](https://miro.medium.com/max/810/1*eNlySI0ufZJweqS4IB7HZA.png "Large example image")

We downloaded the file to our box and then checked the contents of it

![placeholder](https://miro.medium.com/max/573/1*KVdbsdIglWHmZ1vZ0PipiA.png "Large example image")

We have some blobsums so we move on getting each of them to our box

![placeholder](https://miro.medium.com/max/810/1*t-1oc0kGH_N2j5fMJzQOpg.png "Large example image")

Like the above way, we get the blobsum files to our machine

![placeholder](https://miro.medium.com/max/810/1*lKMMJf47qRE56e9rnoexQw.png "Large example image")

We are good to go and since these all files were gzip compressed, we have to unzip and check all of those, for the sake of time we skip that part in this writeup and move onto the main part

![placeholder](https://miro.medium.com/max/810/1*DW0X93ebgAKykDSFvB2yug.png "Large example image")

One of the blobsum folder, we got a file named 01-ssh.sh and looking further into it we saw that it has something which leaks a passphrase to a ssh key, so there might be the ssh key too somewhere, upon looking more

![placeholder](https://miro.medium.com/max/573/1*5D9eOEyeBjNOp6zbWkSxdQ.png "Large example image")

We found the folder containing the ssh keys and also a file named config

![placeholder](https://miro.medium.com/max/362/1*5PMLjU1Fk6arVv8iSqJ76Q.png "Large example image")

We see that the config file reveals the **Username** , **Port** and the **Hostname**, so we connect to the box through SSH

![placeholder](https://miro.medium.com/max/573/1*5RTaPknJZBCyfJCZy80Wkw.png "Large example image")

Putting the passphrase and now we are in so we get our User Flag

![placeholder](https://miro.medium.com/max/280/1*Me8esRQ00xCg9Dy8fQd-jA.png "Large example image")

Moving onto the priv esc part

Privilege Escalation

As we remember from the above part that we had access onto the Bolt webpage, we move onto the **Configuration** option and then move onto changing the ***config.yml*** file

![placeholder](https://miro.medium.com/max/810/1*lo-SgUQIUnQUdLs0p5aJ7w.png "Large example image")

Here we add php to accepted file types list so that we can upload a php file

![placeholder](https://miro.medium.com/max/810/1*EJ0zluhLNKb8lxDoK5ZRng.png "Large example image")

Here we have our file upload functionality and upload our php file

![placeholder](https://miro.medium.com/max/389/1*Sx-kh1jcdDB85xs3VfrjvA.png "Large example image")

Good to go and we upload our file and see

![placeholder](https://miro.medium.com/max/810/1*z1VyO80IIVAgcPNlSkpbZg.png "Large example image")

Our file got uploaded and accessing that

![placeholder](https://miro.medium.com/max/573/1*GJzYEsXEA7Y2SPos6mj2oA.png "Large example image")

We see that it got uploaded but nothing to display as we didnt put our command

![placeholder](https://miro.medium.com/max/573/1*adiC4XjRRcfigV6dtoHw1Q.png "Large example image")

Here we can confirm that our script is running perfectly, but in just like less than 15 seconds

![placeholder](https://miro.medium.com/max/573/1*VfGbJRm4_wpoAGgUh_oPuA.png "Large example image")

We see that our file gets removed automatically and everything gets reset, even the changed in the ***config.yml*** file we made, so we do all the stuff as fast as possible and then redo our bash one liner reverse shell command

![placeholder](https://miro.medium.com/max/573/1*C9XqNbJjvpiBUMVSkR0q-w.png "Large example image")

This bash one liner reverse shell command doesnt work as the webpage cannot reach our IP

![placeholder](https://miro.medium.com/max/573/1*wgK2BP0GXvcTFVUoXkg8yw.png "Large example image")

Here we remote port forwarded to the box and then also do some changed in our reverse shell technique

![placeholder](https://miro.medium.com/max/810/1*vDxBPKB0ynSzrXfrMVDwKg.png "Large example image")

We created a new php reverse shell script and then move onto uploading it

![placeholder](https://miro.medium.com/max/573/1*j5uHZztoapewvZoPqBQAhQ.png "Large example image")

We are set to go and access our php script and looking back into the netcat listener

![placeholder](https://miro.medium.com/max/494/1*odcMToJ2PStucvWNWGxXGQ.png "Large example image")

We got reverse shell successfully

![placeholder](https://miro.medium.com/max/573/1*XUkyaM6uAzMv9seHmd6lPg.png "Large example image")

Looking on the ***sudo -l*** command we see that user www-data can run restic backup command with sudo without password

![placeholder](https://miro.medium.com/max/573/1*QvV6QCyzX954h_g5X6uQkg.png "Large example image")

We created our initial repository by the above command

![placeholder](https://miro.medium.com/max/573/1*Ie31S7YB0EuVJzjt-1h3jw.png "Large example image")

Here we start the **rest-server** on the same directory as the repository created and also the server started on port 8000

![placeholder](https://miro.medium.com/max/573/1*nj7sWLz9tOegGV-SKeiZSg.png "Large example image")

Again remote port forwarding the port 8000 to the box through SSH

![placeholder](https://miro.medium.com/max/573/1*vhvGMc01xkx-1HHxf7ARng.png "Large example image")

Now we ran the command with sudo and created the backup of the /root directory onto our box

![placeholder](https://miro.medium.com/max/810/1*QE5nda9ZGn1Hs3hc8lxk6g.png "Large example image")

We see that the snapshot is created on our local machine by name **snapshots**

![placeholder](https://miro.medium.com/max/810/1*Ykrc8hptWxKbj7sYRXH-kA.png "Large example image")

Now we restored the backup snapshot and then see that we have the contents of the root folder

![placeholder](https://miro.medium.com/max/494/1*AkDr3czbUfnuJw-wN7Og3g.png "Large example image")

We can see the root folder on our box and into that we have the contents

![placeholder](https://miro.medium.com/max/573/1*-sODnMf2A1WaAVGCvpdDLg.png "Large example image")

Here we got the root flag and also we can see that we have .ssh containing the ssh keys for the root user

![placeholder](https://miro.medium.com/max/494/1*awPb-RM1U1VJMGEtGtnFeg.png "Large example image")

So we now connect to the root user through SSH

![placeholder](https://miro.medium.com/max/573/1*_Ez2glLiFFzI7Btn_WHU-w.png "Large example image")

The box is completed as we got complete access of it, hope you enjoyed the writeup

# References

<a href="https://github.com/restic/rest-server">Restic Server</a>





