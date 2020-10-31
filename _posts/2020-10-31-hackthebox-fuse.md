---
date: 2020-10-31 12:26:40
layout: post
title: HackTheBox - Fuse
description: Easy Linux Machine containing Bludit and sudo Security Bypass Privilege Escalation
image: https://cdn-images-1.medium.com/max/800/0*qmMrZ58DcVOsKUCN
optimized_image: https://cdn-images-1.medium.com/max/800/0*qmMrZ58DcVOsKUCN
category: HackTheBox
tags:
  - sudo
  - bludit
  - windows
author: anishka
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">anishka</a>. Also join me on discord.

The IP of this box is **10.10.10.193**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/1200/1*CEXSPgTW0Kfna7JTmImdrQ.png "Large example image")

We have a lot of Open Ports, along with Web Port Open too, so we move onto the web part first

## Web

Checking the IP on the browser

![placeholder](https://cdn-images-1.medium.com/max/1200/1*2BEkpa2M4lMufjLTkXHWBw.png "Large example image")

We see we are unable to use since the IP redirects us to a domain, as we don't have the domain in our hosts file we cant access it

![placeholder](https://cdn-images-1.medium.com/max/1200/1*QFW2k2Db6jhf74p-TvITGg.png "Large example image")

After adding it to the host file, we are able to access the website and we can see that it has ***PaperCut Print Logger*** print logs, clicking on the different datesheet links

![placeholder](https://cdn-images-1.medium.com/max/800/1*LYKNKjbvl_cze1CbgkXApg.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/800/1*wd9e8FFQkBPrQ5tRmfsXnQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/800/1*qNCVSlvW3sjP1y1hmIC4qQ.png "Large example image")

We get few users, so now we keep the username into a file

![placeholder](https://cdn-images-1.medium.com/max/800/1*jXqQ4GaA88PW4KilEA482g.png "Large example image")

Now we use cewl to spider the website and get words for password list since on the website , there were many potential password looking words

![placeholder](https://cdn-images-1.medium.com/max/800/1*jBHNeenZY5sBPDTEpP8Zpg.png "Large example image")

Now we use metasploit's **smb_login** module to bruteforce smb logins

![placeholder](https://cdn-images-1.medium.com/max/800/1*roitsm9k3o7z0aJZoRdP6w.png "Large example image")

Running the module , we see get only 3 successful hits on the same password

![placeholder](https://cdn-images-1.medium.com/max/800/1*Vs_x9QA9byYrk0_ePy6BsA.png "Large example image")

Trying to login with ***smbclient*** with the credentials we got

![placeholder](https://cdn-images-1.medium.com/max/800/1*B8JqcKqWz7jpMu4sPXnalw.png "Large example image")

We see that it doesn't login and returns with a **NT_STATUS_PASSWORD_MUST_CHANGE** error which tells us that we have to first change the password of the user and then login

So we use ***smbpasswd*** to do the same and here we have to be quick doing these steps as the password reverts fastly

![placeholder](https://cdn-images-1.medium.com/max/800/1*P7iKH7I_iDQfDI3Z6uipXQ.png "Large example image")

Now we check the shares of the user ***tlavel*** after changing the password

![placeholder](https://cdn-images-1.medium.com/max/800/1*cpxGP5CZ0MQJh7xKBcylMw.png "Large example image")

We did the same for other 2 users too

![placeholder](https://cdn-images-1.medium.com/max/800/1*5zuOMcnkzfEHLCg590l3bg.png "Large example image")

We got nothing interesting from SMB as we didnt had permissions to access most of them , so I use the new creds to login to RPC using ***rpclient***

![placeholder](https://cdn-images-1.medium.com/max/800/1*TRg9s6KyvufY8l_y1HLqxg.png "Large example image")

Since on the website we saw it was related to printers, we run the command ***enumprinters*** to enumerate printers on RPC

![placeholder](https://cdn-images-1.medium.com/max/800/1*MiT8wL9ywfbibeYeIq4SCw.png "Large example image")

We see we got a password leak here, also we can get more internal users using the ***enumdomusers*** command

![placeholder](https://cdn-images-1.medium.com/max/800/1*NQhuKixOMhUOIMGFx2mNNA.png "Large example image")

We save the additional users on the same file and then bruteforce it for WINRM using ***crackmapexec***

![placeholder](https://cdn-images-1.medium.com/max/800/1*DOH84LRtSbKhKy34Qqd-0Q.png "Large example image")

We got a successful hit and now we use that credentials to login through ***Evil-WinRM***

![placeholder](https://cdn-images-1.medium.com/max/800/1*HFvbVxIttzGNcxk0S4vTlA.png "Large example image")

We got the user and the user flag

## Privilege Escalation

Checking the privileges of the current user

![placeholder](https://cdn-images-1.medium.com/max/800/1*Vp9qXfFRZhkLO32HtZ9flA.png "Large example image")

As you can see the privilege **"SeLoadDriverPrivilege"** is present in the user's access token

At this point we can use the PoC tool ***EOPLOADDRIVER***, which will allow us to:

--> Enable the SeLoadDriverPrivilege privilege
--> Create the registry key under HKEY_CURRENT_USER (HKCU) and set driver configuration settings
--> Execute the NTLoadDriver function, specifying the registry key previously created

The tool can be invoked as shown below:

```EOPLOADDRIVER.exe RegistryKey DriverImagePath```

The RegistryKey parameter specifies the registry key created under HKCU ("Registry User{NON_PRIVILEGED_USER_SID}", while the DriverImagePath specifies the location of the driver in the file system.

![placeholder](https://cdn-images-1.medium.com/max/800/1*VpXUruSc16umiZSAP00V2g.png "Large example image")

We have uploaded the **Capcom.sys** file and our **EOPLOADDRIVER.exe** to load the driver

![placeholder](https://cdn-images-1.medium.com/max/800/1*XeHc8GIMXjvk81h5YNjmTA.png "Large example image")

We ran the command so that it loads the Capcom driver

![placeholder](https://cdn-images-1.medium.com/max/800/1*zZZKQTaQgxKfJfmSP3sLaQ.png "Large example image")

We uploaded our Exploit along with netcat for windows and a batch file which will get triggered and help us get reverse shell

![placeholder](https://cdn-images-1.medium.com/max/800/1*CylOn977eBZPq8MDD8GtLA.png "Large example image")

Now we run the exploit and check our netcat listener

![placeholder](https://cdn-images-1.medium.com/max/800/1*R2H_eER0TjQ52MwvijdQ6w.png "Large example image")

We got shell as SYSTEM and now we get the root flag

![placeholder](https://cdn-images-1.medium.com/max/800/1*x4GsO1qThcgSKsgXCA3C0g.png "Large example image")




# References

<a href="https://www.tarlogic.com/en/blog/abusing-seloaddriverprivilege-for-privilege-escalation/">Abusing SeLoadDriverPrivileges For Privilege Escalation</a>

<a href="https://book.hacktricks.xyz/windows/active-directory-methodology/privileged-accounts-and-token-privileges#seloaddriverprivilege">SeLoadDriverPrivileges</a>



<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










