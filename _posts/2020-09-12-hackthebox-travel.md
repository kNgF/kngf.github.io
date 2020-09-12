---
date: 2020-09-12 12:26:40
layout: post
title: HackTheBox - Travel
description: Hard Linux Machine containing Memcached SSRF
image: https://cdn-images-1.medium.com/max/800/0*rZUHeB3FCqD9_IVX
optimized_image: https://cdn-images-1.medium.com/max/800/0*rZUHeB3FCqD9_IVX
category: HackTheBox
tags:
  - SSRF
  - linux
author: anishka
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">anishka</a>. Also join me on discord.

The IP of this box is **10.10.10.189**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/800/1*8315r_aJ3r9fTWfxu93ZDQ.png "Large example image")

We have 3 Open Ports , one for SSH and the other two for Web

Moving directly towards the web part

## Web

![placeholder](https://cdn-images-1.medium.com/max/1200/1*oM_2GsNwWhZGocOL5AXF-w.png "Large example image")

We see a good webpage, but nothing much to see and find here , so moving towards the https version

![placeholder](https://cdn-images-1.medium.com/max/800/1*aTpDzqgp5Ihk0cfhT9SDZQ.png "Large example image")

We have a message by the admin the SSL is not yet implemented on the server and tell us to use the non-SSL websites

![placeholder](https://cdn-images-1.medium.com/max/800/1*wRNayYzX5DmNbW2hiWzVog.png "Large example image")

Checking the certificate, we get one domain ***www.travel.htb***

![placeholder](https://cdn-images-1.medium.com/max/800/1*Zf-xitEwkYElpB-QOmpEAQ.png "Large example image")

Looking further more into the certificate, we get more domains which we put on our hosts file

![placeholder](https://cdn-images-1.medium.com/max/800/1*5OISritxi8unCpCe6qWowA.png "Large example image")

Since we found many subdomains, I ran ***wfuzz*** to bruteforce more potential subdomains and got 2 more which were added to hosts file as well

![placeholder](https://cdn-images-1.medium.com/max/800/1*eHcV4Fcc65h-SatNF4U66A.png "Large example image")

Checking the ***ssl.travel.htb*** domain, it returned the same webpage as the SSL version of Web before

![placeholder](https://cdn-images-1.medium.com/max/1200/1*l7FPRqHSqcv-BYpGSeIM2A.png "Large example image")

On the ***blog.travel.htb*** domain, we get a **wordpress** blog and nothing much interesting to find here

![placeholder](https://cdn-images-1.medium.com/max/1200/1*Hj9PXJ8fgzoG_G_QQwd4_Q.png "Large example image")

On the other hand on ***blog-dev.travel.htb***, we get a forbidden error message so we run gobuster against it to find potential point of interests

![placeholder](https://cdn-images-1.medium.com/max/800/1*RjjV-hmNGiBtpX64o3333w.png "Large example image")

We found a git directory , so we use a tool to dump the the git repository 

> gitdumper.sh http://blog-dev.travel.htb/.git/ ~/htb/travel/repo

![placeholder](https://cdn-images-1.medium.com/max/800/1*WygcwMQwNHawKFlV0vxiKA.png "Large example image")

We downloaded the repository, we also need to extract it using a tool, the command is given below

> extractor.sh ~/htb/travel/repo ~/htb/travel/repo-extract

![placeholder](https://cdn-images-1.medium.com/max/800/1*XOALWyX2oQmNszzXOjuzeQ.png "Large example image")

Checking the contents, we found few files in the ***git*** repository and then checked some of the files, one of them was ***rss_template.php***, we see that it is using **SimplePie** and that is querying **memcache** server which also has the prefix xct_

Checking more into the code below

![placeholder](https://cdn-images-1.medium.com/max/1200/1*2ygY_7-7o9uyBtDZ7FBbWg.png "Large example image")

We see that there is a GET request with debug parameter

![placeholder](https://cdn-images-1.medium.com/max/800/1*aMet5TfWPx7Lt_NPeIxcNg.png "Large example image")

Checking the template.php file, we see that there is a class **TemplateHelper** and inside there are functions **_construct and _wakeup**

So we know that *Awesome RSS* was in the ***blog.travel.htb*** domain, so we now run gobuster against the domain

![placeholder](https://cdn-images-1.medium.com/max/800/1*RJLKNFCeoeHNfr9BhFBLlQ.png "Large example image")

We see that is has a ***/rss*** directory , which redirects us to ***/awesome-rss*** directory

![placeholder](https://cdn-images-1.medium.com/max/1200/1*lh1JFlAEch2g5ndrVttOkg.png "Large example image")

Now we intercept the request and then test the debug parameter which we saw in the code, we can see the comment in the code that it has a PHP serialized object

![placeholder](https://cdn-images-1.medium.com/max/1200/1*CZyXnFpm_qo0uIZ7RIv65g.png "Large example image")

If we see the code of SimplePie from the github directory and also check the memcache section in ***library/SimplePie/Cache/Memcache.php*** we have:

`$this->name = $this->options['extras']['prefix'] . md5("$name:$type");`

and **$name** was set to:

`call_user_func($this->cache_name_function, $url)`

The **cache_name_function** callback is defined in ***library/SimplePie.php*** as md5():

`public function set_cache_name_function($function = 'md5')`

so we have to take the md5sum of the URL twice with the **TYPE_FEED**, i.e, ***"spc"***

![placeholder](https://cdn-images-1.medium.com/max/800/1*GBdSTHaA2LI1LkJWuzDZ_g.png "Large example image")

We already saw the URL already in the ***rss_template.php*** code

![placeholder](https://cdn-images-1.medium.com/max/800/1*-WRunRzQREonX9BRme1SXw.png "Large example image")

Now we convert our URL with **spc** to MD5 hash and now we created our PHP file

![placeholder](https://cdn-images-1.medium.com/max/800/1*0ipm0oM54Tj5S9_f62IZQw.png "Large example image")

Our code is ready where we used the same class from the ***template.php*** code to create our serialized object

![placeholder](https://cdn-images-1.medium.com/max/800/1*GkMeF_atAJauYX9vFVTV8g.png "Large example image")

We got our PHP serialized code which we will use on the on the memcache server to set the key and value

![placeholder](https://cdn-images-1.medium.com/max/800/1*9VDaNwBgm62mYqqF2UM6vQ.png "Large example image")

I used CyberChef to manage the URL and the memcache commands and then URL encode them

![placeholder](https://cdn-images-1.medium.com/max/1200/1*cH59RsI8IBMradd0f_OyjQ.png "Large example image")

We used curl to process the request

![placeholder](https://cdn-images-1.medium.com/max/1200/1*5WKbenSCqlA1XwTx_Pf8fQ.png "Large example image")

Since we don't know the location where our shell got uploaded

![placeholder](https://cdn-images-1.medium.com/max/800/1*xRi0jfsVgdudSntFswk57A.png "Large example image")

If we looked onto the **README.md** file on the git repository extracts, it tells us that it copies ***rss_template.php and template.php*** to **/wp-content/themes/twentytwenty** and also creates a **logs** directory on that location , so that might be the location of our shell upload

![placeholder](https://cdn-images-1.medium.com/max/800/1*eA6gcvepOSxg5gUYeFlf4A.png "Large example image")

We found our shell and got command execution successfully

![placeholder](https://cdn-images-1.medium.com/max/800/1*q_9wNOl3kLPS9JTvsrwHBg.png "Large example image")

We get reverse shell as ***www-data*** and now moving into user privilege escalation

![placeholder](https://cdn-images-1.medium.com/max/800/1*LckWgwtMuOeGl-UIV4SKiw.png "Large example image")

Checking the ***/opt*** folder , we have a **wordpress** directory and inside of it we find a SQL backup file

![placeholder](https://cdn-images-1.medium.com/max/800/1*l2QVVZ5ASc-ocuvI_LmIFA.png "Large example image")

We found a hash for user ***lynik-admin*** and now use **hashcat** to crack it

![placeholder](https://cdn-images-1.medium.com/max/800/1*sHKdRBq7p9mW3L-LT6A8tA.png "Large example image")

We cracked the password and now connect to the machine as user ***lynik-admin*** through SSH

![placeholder](https://cdn-images-1.medium.com/max/800/1*xOW_jdD1cqIwfphtB1A8Pg.png "Large example image")

## Privilege Escalation

Checking the user's directory, we find two unusual files ***.ldaprc*** and ***.viminfo***

![placeholder](https://cdn-images-1.medium.com/max/800/1*BuWubQzmUvoM5sKdvGihaA.png "Large example image")

Checking the ***.ldaprc*** file, it contains the ldap details

![placeholder](https://cdn-images-1.medium.com/max/800/1*a7zKlEVxccWJ_UCfCPXTSA.png "Large example image")

On the other hand, checking on the ***.viminfo*** file, we get a password ***Theroadlesstraveled*** which might be the password for ldap

![placeholder](https://cdn-images-1.medium.com/max/800/1*KsEIwO45RLE_5nROidzNxQ.png "Large example image")

Doing a ldapsearch query with the password we got

![placeholder](https://cdn-images-1.medium.com/max/800/1*_9Ip0AIcB2PknnxP8gO0jw.png "Large example image")

We see many ldap users and the list keeps going on

![placeholder](https://cdn-images-1.medium.com/max/800/1*3Z2SidEA686owM_M32OAPQ.png "Large example image")

We see that all of the users have a fixed ***gidNumber*** ,i.e, **5000**, we can change that to **27** which is the default group ID for sudo, doing this will help us use sudo with any program

![placeholder](https://cdn-images-1.medium.com/max/1200/1*y-Q-YXwwFZMwl5xtUpp8ig.png "Large example image")

Since ldap entries are modified using a **.ldif** file, we created our file with details to modify and also added a ssh key so that can connect to that user through SSH first

![placeholder](https://cdn-images-1.medium.com/max/800/1*uM6arixktrOClKEds3Nmlw.png "Large example image")

We used ***ldapmodify*** command to modify the ldap entry

![placeholder](https://cdn-images-1.medium.com/max/800/1*5UVaE9GJlDPJcjNZZZcZ0w.png "Large example image")

We can confirm that the entry was modified and now connect to the user through SSH

![placeholder](https://cdn-images-1.medium.com/max/800/1*9SB-UtSMfiywFdxMrnNsMg.png "Large example image")

We got in as the user ***jerry*** and can confirm that the user jerry is in ***sudoer's*** group

![placeholder](https://cdn-images-1.medium.com/max/800/1*edPh6ik7jI7QAfz0NNc4Hg.png "Large example image")

Now running ***sudo -l*** command, it asks for password , which we dont know

![placeholder](https://cdn-images-1.medium.com/max/800/1*C2F_nTEzJzkxLmyQ7xj84w.png "Large example image")

We have to modify the entry of other user again and this time add a user password too so that we can use it with sudo

![placeholder](https://cdn-images-1.medium.com/max/1200/1*U6nWhyCkc5uJVD7xkVJmFg.png "Large example image")

We switched to user ***brian*** this time with the same SSH key and now use sudo with bash

![placeholder](https://cdn-images-1.medium.com/max/800/1*01xUekPjHlWmCNvyjvCc9A.png "Large example image")

We got root and the root flag

![placeholder](https://cdn-images-1.medium.com/max/800/1*eZmZvv1QgbwmO5SRp-xxdg.png "Large example image")


# References

<a href="https://www.digitalocean.com/community/tutorials/how-to-use-ldif-files-to-make-changes-to-an-openldap-system">How to Use LDIF Files to Change LDAP System</a>

<a href="https://docs.oracle.com/cd/E37838_01/html/E61025/ssh-ldappubkeytask.html">How to Set Up Secure Shell User Authentication From Public Keys Stored in LDAP</a>

<a href="https://www.ibm.com/support/knowledgecenter/SS4T7T_6.0.1/com.ibm.help.seas.secure.doc/seas_entries_for_ssh_public_key_in_the_ldap_server.html">
Entries for SSH Public Key in the LDAP Server
</a>


<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










