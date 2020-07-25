---
date: 2020-07-25 12:00:00
layout: post
title: HackTheBox - Cascade
description: Medium Level Active Directory Environment
image: https://cdn-images-1.medium.com/max/716/0*B2BENjX5ZywnLP7X
optimized_image: https://cdn-images-1.medium.com/max/716/0*B2BENjX5ZywnLP7X
category: HackTheBox
tags:
  - hacking
  - Active Directory
  - Penetration Testing
author: hulegu
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">hulegu</a>. Also join me on discord.

The IP of this box is **10.10.10.182**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/716/1*1eSTkjgA7s09Ijv9cP-wDg.png "Large example image")

We get alot of Open Ports and looking from the ports, we see its an Active Directory environment

# Enum4Linux

Running the enum4linux tools

![placeholder](https://cdn-images-1.medium.com/max/1075/1*oojhj1B8rvNr8xPRATEnrA.png "Large example image")

Here we have the users and also the groups down below

![placeholder](https://cdn-images-1.medium.com/max/1075/1*_XV4k4eZqOQuoSP48gkjEg.png "Large example image")

We put the users in a separate *txt* file

![placeholder](https://cdn-images-1.medium.com/max/716/1*hCHVtAnux_JiXjFm1Z4ELg.png "Large example image")

Now we move onto doing some ldap queries

# LDAP

> ldapsearch -x -b "dc=cascade,dc=local" -H ldap://10.10.10.182

![placeholder](https://cdn-images-1.medium.com/max/716/1*4tnnmaAwC0gl3aAn1gnFpQ.png "Large example image")

We get alot of results from the query, but looking onto it carefully or either greping for potential password or more usernames

![placeholder](https://cdn-images-1.medium.com/max/716/1*xV9PqIJTIpq5K458BYBvow.png "Large example image")

We see something like ***cascadeLegacyPwd*** and it has something base64 encoded string , by decoding it

![placeholder](https://cdn-images-1.medium.com/max/716/1*dAOuLic78_iyafo3XC2Q4A.png "Large example image")

We see its decodes to ***rY4n5eva***

Since we see that **wsman** is open from NMAP

![placeholder](https://cdn-images-1.medium.com/max/716/1*bsQQqPjcNwNE1oKITQcSUw.png "Large example image")

We can try **Evil-WinRM** to login to the usernames we have

# SMB

> crackmapexec smb 10.10.10.182 -u users -p 'rY4n5eva'

![placeholder](https://cdn-images-1.medium.com/max/1075/1*Jv81Gs8EnrnOc_d-1uNKmA.png "Large example image")

We see that the user ***r.thompson*** get a hit , so we move onto getting the SMB shares of user ***r.thompson***

![placeholder](https://cdn-images-1.medium.com/max/716/1*xED3w81VnBhw0BkaZLfAZA.png "Large example image")

We get alot of shares , the one which is useful here is **Data**

![placeholder](https://cdn-images-1.medium.com/max/716/1*ImHWBBx0iMBtPMr62AU9ww.png "Large example image")

We logged into the SMB share Data and now we try to get all the files which we can

![placeholder](https://cdn-images-1.medium.com/max/716/1*IvUnKQ2o_Y56pqfgcwRDig.png "Large example image")

The **IT** directory was only accessible to us for now, so we get each and every files from every folders inside it

![placeholder](https://cdn-images-1.medium.com/max/716/1*vc4A2GUg1K9vgjYds5QZsg.png "Large example image")

Here are the 4 files we got , looking upon the **Meeting Notes html** file, we see

![placeholder](https://cdn-images-1.medium.com/max/1075/1*NWu3B3dRsUd6Hn8Do0oD6g.png "Large example image")

We see that it says that it has created a user ***TempAdmin*** whose password will be same as the administrator account password

![placeholder](https://cdn-images-1.medium.com/max/716/1*Z4GBNUBDKbNiR9AuFM8yAw.png "Large example image")

Checking the contents of **VNC Install.reg** file, we see that it has password in hex, we can decrypt it using a tool method which I found on google

![placeholder](https://cdn-images-1.medium.com/max/716/1*9sHf0XZjTz94iPTDn2ytAQ.png "Large example image")

We decrypted the password for user ***s.smith*** which we can confirm using **crackmapexec** too

![placeholder](https://cdn-images-1.medium.com/max/1075/1*yBGTMauDX_LVtm87Gj058w.png "Large example image")

Since we got a hit on WinRM for that user , we use **Evil-WinRM** to login

![placeholder](https://cdn-images-1.medium.com/max/716/1*xDRVLa1kh0RkP2xJQmuptQ.png "Large example image")

We got logged in successfully and now we move onto getting user flag

![placeholder](https://cdn-images-1.medium.com/max/716/1*sQd7qYbsyaxi_7drSF8icA.png "Large example image")

# Privilege Escalation 

Also checking the SMB shares enabled for user ***s.smith***

![placeholder](https://cdn-images-1.medium.com/max/716/1*tIuDKLr1EhF0WumquFDj_w.png "Large example image")

We didnt had access to **Audit$** share with user ***r.thompson*** , but with user ***s.smith***, we do have access to that

![placeholder](https://cdn-images-1.medium.com/max/716/1*_kZ4qyOjJ_G1KXowc5CYjw.png "Large example image")

Inside of the shares , we see some binaries , dlls and a DB file so we get them all to our local machine

![placeholder](https://cdn-images-1.medium.com/max/1075/1*EW_sJiNzLD8tk_FKlB3JjA.png "Large example image")

Checking the batch file code , we see it runs the **CascAudit.exe** binary with the DB file, so we just check the DB file using ***sqlite3***

![placeholder](https://cdn-images-1.medium.com/max/1075/1*YZKPiLIsTFibaZGH6lO_LQ.png "Large example image")

Upon checking and getting all the information , we see that it has a username ***ArkSvc*** and password in base64

![placeholder](https://cdn-images-1.medium.com/max/716/1*CvLcWj_TaMJZhkL5wT26GA.png "Large example image")

When decoding the base64 string , we see that it is still encrypted in some form

![placeholder](https://cdn-images-1.medium.com/max/716/1*UDAK_fV-SqlIzYtNsqLctw.png "Large example image")

Reversing the ***CascAudit.exe*** binary and ***CascCrypto dll***

![placeholder](https://cdn-images-1.medium.com/max/1075/1*EjAySfju3URHUk8kVDVBdQ.png "Large example image")

We see that it uses a key for decryption , looking further onto the reversed dll

![placeholder](https://cdn-images-1.medium.com/max/1075/1*v2En0vwraelyZ_Kkb6wuBw.png "Large example image")

We see that the function **EncryptString** uses a string and a key which we saw above and also from the code we can see its using **AES** encryption , so we can just use **Cyber Chef** to do the decryption

![placeholder](https://cdn-images-1.medium.com/max/716/1*o27TW-SyNyMZKbbeCZ7RGw.png "Large example image")

We got the password decrypted for user ***ArkSvc***

![placeholder](https://cdn-images-1.medium.com/max/716/1*mFa2BpWcykA_msOGWzOTCA.png "Large example image")

Checking the group permissions of the current user

![placeholder](https://cdn-images-1.medium.com/max/1075/1*FeTEnZlAogB9JtmogAZVFA.png "Large example image")

We see that it has group permissions for AD Recycle Bin which contains the deleted objects of the AD and also we can restore that objects if it is enabled

From the below command we can see the Deleted Objects on the AD Recycle Bin

> Get-ADObject -ldapFilter:"(msDS-LastKnownRDN=*)"  -IncludeDeletedObjects

![placeholder](https://cdn-images-1.medium.com/max/716/1*Sz0KYtRq13T1KRHqoijSTA.png "Large example image")

We see the user ***TempAdmin*** which was also on a note before saying that it has the same password as administrator account, so we can try to get its password using listing its properties

> Get-ADObject -Filter {displayName -eq "TempAdmin"} -IncludeDeletedObjects -Property *

![placeholder](https://cdn-images-1.medium.com/max/716/1*qKxJXroumS1YhJuo8D0JkQ.png "Large example image")

We got the password in Base64 string format and now decode it

![placeholder](https://cdn-images-1.medium.com/max/716/1*tEkNZy883r0dbS-V5bgUoQ.png "Large example image")

Logging into the administrator account using the creds

![placeholder](https://cdn-images-1.medium.com/max/716/1*dShqrkzqh0kQlzmH6rqgjw.png "Large example image")

We got Administrator access to the domain and now we can get the root flag

![placeholder](https://cdn-images-1.medium.com/max/716/1*a_7PIJSvCqAyMXdpJRdSww.png "Large example image")


# References 

<a href="https://www.lepide.com/how-to/restore-deleted-objects-in-active-directory.html">A Step-By-Step Guide to Restore Deleted Objects in Active Directory</a>

<a href="https://github.com/frizb/PasswordDecrypts">Password Decrypts</a>

<a href="https://devconnected.com/how-to-search-ldap-using-ldapsearch-examples/">LDAPSearch Tutorial</a>

<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 
