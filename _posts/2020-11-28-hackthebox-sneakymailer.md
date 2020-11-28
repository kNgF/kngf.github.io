---
date: 2020-11-28 12:26:40
layout: post
title: HackTheBox - SneakyMailer
description: Medium Linux Machine where we have phishing, exploiting PyPi and then exploiting sudo privileges on Pip3
image: https://miro.medium.com/max/700/0*cmy0Jaa5IIhYSc4V
optimized_image: https://miro.medium.com/max/700/0*cmy0Jaa5IIhYSc4V
category: HackTheBox
tags:
  - linux
  - medium
author: feodore
paginate: true
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">feodore</a>. Also join me on discord.

The IP of this box is **10.10.10.197**

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://miro.medium.com/max/1000/1*dGfklsG7kj91ChGDvDTOVA.png "Large example image")

We get a lot of Open Ports, starting off with the web part

## Web

Checking the IP in the browser redirects us to a domain

![placeholder](https://miro.medium.com/max/1000/1*j15uttWRQpvZhLtLYO-pfw.png "Large example image")

Adding it to our hosts file and checking again

![placeholder](https://miro.medium.com/max/1000/1*OYSqMX_wYKmY5LLMf_ClUw.png "Large example image")

We land to the dashboard of **Sneaky Corp**, clicking on the **Team** section

![placeholder](https://miro.medium.com/max/1000/1*nyH-m6Btqz6GnYAKm5wHeQ.png "Large example image")

We get a lot of emails, since the box had **SMTP** enabled, we can potentially try phishing against all the emails

![placeholder](https://miro.medium.com/max/700/1*txsLwIvCYXxbxzQgIH7VuA.png "Large example image")

We used ***swaks*** to do the task with the help of simple bash scripting so that it tries with each email addresses and now we wait on our netcat listener to get a hit back

![placeholder](https://miro.medium.com/max/700/1*UQA_iyWFyWjVgXWeT_FA7w.png "Large example image")

We see we got a response and in the response we have parameters which leaks the password but it is in URL encoded for so we decode it and get the password

![placeholder](https://miro.medium.com/max/700/1*7qpdA608i0cp_a9hzQF2yQ.png "Large example image")

We now login to the mail using ***Evolution*** and check the **Sent Items**

![placeholder](https://miro.medium.com/max/1000/1*hLYGJKVnoe3sP1-a5bTHVg.png "Large example image")

We have 2 mails here, checking both of them

![placeholder](https://miro.medium.com/max/1000/1*4yU1ff-r7L-f6vbjYniAYA.png "Large example image")

One of the mail has credentials and the other one has a email sent about installing modules in the PyPI service

Testing the credentials into **FTP**

![placeholder](https://miro.medium.com/max/288/1*vGYGdyeUqT0XMzklHm4f5Q.png "Large example image")

We logged into FTP successfully and now we move onto checking the contents inside it

![placeholder](https://miro.medium.com/max/468/1*-yG8vyn4245N0ZqEqHz5Og.png "Large example image")

Also checking the vhosts we having on this box

![placeholder](https://miro.medium.com/max/700/1*HbnN9EoziwYNZGYIQLLarg.png "Large example image")

We got hit on ***dev.sneakycorp.htb*** and we add it to our hosts file and run it

![placeholder](https://miro.medium.com/max/1000/1*xEUIA34zDQKvFzal0kOXng.png "Large example image")

We see it being the same as the main domain with only few changes, we upload a web shell through FTP and try to access it

![placeholder](https://miro.medium.com/max/587/1*LO6oLrfF4R3E14qb8Ogonw.png "Large example image")

We got shell successfully and now get a reverse shell and check the netcat listener

![placeholder](https://miro.medium.com/max/587/1*Jj1lq0GDaLx2jMNYS4kIOQ.png "Large example image")

We can switch to developer user using the password which we got through the mail

![placeholder](https://miro.medium.com/max/454/1*wO2PrXDNh_8Wq7vd33Bv0g.png "Large example image")

Checking the webroot, we get a folder pointing to ***pypi.sneakycorp.htb*** subdomain

![placeholder](https://miro.medium.com/max/442/1*ritqJAEW-fHUIsk8G8gfVQ.png "Large example image")

Inside the folder we have a ***.htpasswd*** folder which is readable by any user and inside of it contains the credentials for ***pypi***

![placeholder](https://miro.medium.com/max/442/1*ElayANBLFSKeZOB1NG3OAw.png "Large example image")

Using ***john*** , we crack the password successfully

![placeholder](https://miro.medium.com/max/700/1*xrjx4DV5EYoD4BwcAoOrgA.png "Large example image")

Now we start creating the python package,

1. Create a **~/.pypirc** file with content:

![placeholder](https://miro.medium.com/max/319/1*LGi0wK-C7vWAQf5wkuH3IA.png "Large example image")

2. Install ***whell*** and ***twine***:

```sudo -H pip install wheel twine```

3. Create dirs/files structure:

```
/pkg
  /pkg/example_pkg
  /pkg/example_pkg/__init__.py
  /pkg/setup.py
```

![placeholder](https://miro.medium.com/max/700/1*V-peDdabiRINlc-iFTQJtg.png "Large example image")

4. In ***__init__.py*** can be anything but in ***setup.py*** should be **"installing script"** with reverse shell like this:

![placeholder](https://miro.medium.com/max/700/1*oHejvkzQWJpg_FdwokIzDQ.png "Large example image")

5. Then build a package: ```python setup.py sdist bdist_wheel```

![placeholder](https://miro.medium.com/max/700/1*netE84RmSk4bLywTsLuFwA.png "Large example image")

6. Upload package which should be immediately installed:

 ```python -m twine upload — repository mypypi dist/*```

7. Run a nc listener

![placeholder](https://miro.medium.com/max/700/1*xQlkaRUUTT2Wei6CwRjs5w.png "Large example image")

We got shell as user ***low*** and now can get the user flag

![placeholder](https://miro.medium.com/max/487/1*H5Bzr-BzRhNhigLuEnMaMA.png "Large example image")


## Privilege Escalation

Running the sudo -l command

![placeholder](https://miro.medium.com/max/700/1*AJjhge0Hs4q3HyVDYTCIZw.png "Large example image")

Now we follow the traditional **GTFOBin**’s method and do the steps one by one

![placeholder](https://miro.medium.com/max/700/1*hAQ7qVaMNKJfaYaYd0hKEA.png "Large example image")

We got root!!!



# References

<a href="https://packaging.python.org/tutorials/packaging-projects/">Packaging Python Projects</a>

<a href="https://gtfobins.github.io/gtfobins/pip/#shell">pip3 GTFOBins</a>



<img src="http://www.hackthebox.eu/badge/image/7404" alt="Hack The Box"> 










