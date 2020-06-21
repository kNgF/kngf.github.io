---
date: 2020-06-20 22:55:45
layout: post
title: HackTheBox - ServMon
description: Resolute is an easy Windows Machine
image: https://miro.medium.com/max/1400/0*LfY-rwIbD7fxHt8G
optimized_image: https://miro.medium.com/max/1400/0*LfY-rwIbD7fxHt8G
category: HackTheBox
tags:
  - hacking
  - ctf
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a> . Also join me on discord.
The IP of this box is 10.10.10.184

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/2000/1*9RfXwzDDA_RofTrBbnw3gA.png "Large example image")

We get alot of Open Ports, interesting ones contain Port 21, 22, and 80

We see that can connect through FTP using ***anonymous*** login

![placeholder](https://miro.medium.com/max/1400/1*FTRwp0WL62mwqyBKjvGsIw.png "Large example image")

We got 2 txt files from two folders named Nathan and Nadine

Moving onto the web part, we get a login page

![placeholder](https://miro.medium.com/max/2000/1*rl1Dzu6E39a8wgru-3g28w.png "Large example image")

Also we see that the web is running NVMS-1000, looking for potential exploits on searchsploit

![placeholder](https://miro.medium.com/max/2000/1*M7Sxde1F7ChNTDVnB1LHYw.png "Large example image")

We see that we have a Directory Traversal on it , looking futher into the exploit file

![placeholder](https://miro.medium.com/max/2000/1*ObGo6WYovXbECTIKTx4-Dw.png "Large example image")

We can intercept the request in Burp and modify the request to get the path traversal

![placeholder](https://miro.medium.com/max/1400/1*eRl7r2jBKf7WwVh11DfEGA.png "Large example image")

We see in the response that we get our path traversal, so we move onto checking the text files we got from the FTP

![placeholder](https://miro.medium.com/max/2000/1*WjU0aL6y3naqTIoUeLvrGQ.png "Large example image")

We see that it says Nadine has stored a file named **Passwords.txt** in the Desktop of Nathan

We can try to grab it from the path traversal

![placeholder](https://miro.medium.com/max/1400/1*NE-oOAY4LwgtUDt9FF7G1Q.png "Large example image")

We get a lot of Passwords in return in the response field, we copy it to a text file in our local machine and then try each one of them with both users through SSH

![placeholder](https://miro.medium.com/max/2000/1*Ci_ILsewnZUEO_XxZOpLzg.png "Large example image")

After trying each and every password with both users, we see that L1k3B1gBut7s@W0rk worked for user Nadine and we got connected through SSH successfully

![placeholder](https://miro.medium.com/max/1052/1*caV5aqP1Q7uQCHJNeRVC6w.png "Large example image")

We got the user flag here and now time for privilege escalation

## Privilege Escalation

From the notes file which we got from the **FTP** , we see that the user had setup ***NSClient++*** which we confirm by looking into **Program Files**

![placeholder](https://miro.medium.com/max/1400/1*3RL6T3GSjxCtJ5j17gsvBQ.png "Large example image")

We now get the web password for NSClient++ down below

![placeholder](https://miro.medium.com/max/1400/1*tVjhqUTxPMSM5jrPmYMTLg.png "Large example image")

Now we try to access the web client of NSClient++

![placeholder](https://miro.medium.com/max/2000/1*it6M1cf7YQcIP2fXV0pjEQ.png "Large example image")

Unfortunately, the web GUI was very unstable and unreachable, since we saw it was running on Port **8443**

![placeholder](https://miro.medium.com/max/1400/1*ciLxQM7ZH_nwX4Sw40NghQ.png "Large example image")

We can just Port Forward it to our localhost using SSH

![placeholder](https://miro.medium.com/max/1400/1*bA_m3QDadM_oCcTB2xLyFw.png "Large example image")

Now we try to access the GUI and see it asks us for password, which is the one which we got above

![placeholder](https://miro.medium.com/max/2000/1*GQoCeD8JT2D9B6_Kt7HrTw.png "Large example image")

We have to make sure that the modules **CheckExternalScripts** and **Scheduler** both are enabled

![placeholder](https://miro.medium.com/max/2000/1*npOTkftH6NJentgSP0ifdw.png "Large example image")

Now we upload netcat to the temp folder so that we can use it to get reverse shell

![placeholder](https://miro.medium.com/max/1194/1*qrLPh2W6AkOsbHBUvZwAkg.png "Large example image")

Now we create and add our script using the API like down below

![placeholder](https://miro.medium.com/max/2000/1*wR-zsiL3g0nTI1CynMDXcw.png "Large example image")

We can confirm that our script has been uploaded successfully

![placeholder](https://miro.medium.com/max/1400/1*WeG9v80YCGYyh65uO0k7sg.png "Large example image")

Now we just make a query so that it triggered our script and we get reverse shell

![placeholder](https://miro.medium.com/max/1400/1*ese0dcpirM9It01GuZxIjw.png "Large example image")

Looking back to our netcat listener

![placeholder](https://miro.medium.com/max/1118/1*b3j2NbnTUnW_2gBVhQ7tTg.png "Large example image")

We got shell as NT Authority\System and now we can get the root flag

![placeholder](https://miro.medium.com/max/1048/1*uvo7pFBSi2DqxW9kkveygg.png "Large example image")

# References

<a href="https://docs.nsclient.org/api/rest/">NSClient++ API</a>


<a href="https://www.exploit-db.com/exploits/47774">NVMS 1000 Exploit</a>














