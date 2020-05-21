---
date: 2020-05-09 12:26:40
layout: post
title: HackTheBox - Obscurity
subtitle: HackTheBox Writeup - Obscurity
description: Obscurity is a box which is completely based on python codes where we exploit them one by one to get multiple users and also the root.
image: https://cdn-images-1.medium.com/max/1600/0*UJ7LCAnvHyqJrswx
optimized_image: https://cdn-images-1.medium.com/max/1600/0*UJ7LCAnvHyqJrswx
category: HackTheBox
tags:
  - hackthebox
  - python
  - hacking
  -  penetration testing
author: ferllen
---

Hello Guys , I am <a href="https://twitter.com/_kNgF">Faisal Husaini</a>. My username on HTB is <a href="https://www.hackthebox.eu/home/users/profile/7404">ferllen</a> . Also join me on discord.
The IP of this box is 10.10.10.168

## Port Scan

Running nmap full port scan on it , we get

![placeholder](https://cdn-images-1.medium.com/max/1600/1*XPRXOljrrQSQZQKBVO-51g.png "Large example image")

We get 2 Open Ports and 2 Closed Ports, so we now run service scan for each one of those open ports

![placeholder](https://cdn-images-1.medium.com/max/2400/1*XdFF3IfDDUEB8xo5cDOxJw.png "Large example image")

So on the Open Ports we see that we have one for SSH and other for Web Part on 22 and 8080 respectively. I still don't know why NMAP gave results for Closed Ports
Moving onto the web part

## Port 8080 - Web

Loading the website in the browser with the port

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see that its a webpage related to security , scrolling further more

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")


We see we have a Development section and also we a message to the server devs that the source code of the web server running is in a secret development directory by the filename <em><strong>"SuperSecureServer.py"</strong></em>

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

So we run a simple bash script to automate the bruteforcing of directory search and then we get a successful hit , i.e, develop
Now trying to access the source code

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see its a long code, but reading carefully each functions, we get one which looks interesting

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

On the function ***serveDoc***, we see that it tries to use the exec function on the path, so I just copy some lines of the function to my local machine and try to exploit it manually first

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

These lines were copied and now we execute our python script

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

We see that we can successfully perform code execution by just escaping with a semicolon followed by the ***os.system*** function which will then be followed by our system commands

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

So here we try to get reverse shell and looking back to the netcat listener

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")

![placeholder](https://cdn-images-1.medium.com/max/2400/1*X2j336pCvT5TCX6odfZeHQ.png "Large example image")








> Curabitur blandit tempus porttitor. Nullam quis risus eget urna mollis ornare vel eu leo. Nullam id dolor id nibh ultricies vehicula ut id elit.

Etiam porta **sem malesuada magna** mollis euismod. Cras mattis consectetur purus sit amet fermentum. Aenean lacinia bibendum nulla sed consectetur.

## Inline HTML elements

HTML defines a long list of available inline tags, a complete list of which can be found on the [Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/HTML/Element).

- **To bold text**, use `<strong>`.
- *To italicize text*, use `<em>`.
- Abbreviations, like <abbr title="HyperText Markup Langage">HTML</abbr> should use `<abbr>`, with an optional `title` attribute for the full phrase.
- Citations, like <cite>&mdash; Thiago Rossener</cite>, should use `<cite>`.
- <del>Deleted</del> text should use `<del>` and <ins>inserted</ins> text should use `<ins>`.
- Superscript <sup>text</sup> uses `<sup>` and subscript <sub>text</sub> uses `<sub>`.

Most of these elements are styled by browsers with few modifications on our part.

# Heading 1

## Heading 2

### Heading 3

#### Heading 4

Vivamus sagittis lacus vel augue rutrum faucibus dolor auctor. Duis mollis, est non commodo luctus, nisi erat porttitor ligula, eget lacinia odio sem nec elit. Morbi leo risus, porta ac consectetur ac, vestibulum at eros.

## Code

Cum sociis natoque penatibus et magnis dis `code element` montes, nascetur ridiculus mus.

```js
// Example can be run directly in your JavaScript console

// Create a function that takes two arguments and returns the sum of those arguments
var adder = new Function("a", "b", "return a + b");

// Call the function
adder(2, 6);
// > 8
```

Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa.

## Lists

Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus. Aenean lacinia bibendum nulla sed consectetur. Etiam porta sem malesuada magna mollis euismod. Fusce dapibus, tellus ac cursus commodo, tortor mauris condimentum nibh, ut fermentum massa justo sit amet risus.

* Praesent commodo cursus magna, vel scelerisque nisl consectetur et.
* Donec id elit non mi porta gravida at eget metus.
* Nulla vitae elit libero, a pharetra augue.

Donec ullamcorper nulla non metus auctor fringilla. Nulla vitae elit libero, a pharetra augue.

1. Vestibulum id ligula porta felis euismod semper.
2. Cum sociis natoque penatibus et magnis dis parturient montes, nascetur ridiculus mus.
3. Maecenas sed diam eget risus varius blandit sit amet non magna.

Cras mattis consectetur purus sit amet fermentum. Sed posuere consectetur est at lobortis.

Integer posuere erat a ante venenatis dapibus posuere velit aliquet. Morbi leo risus, porta ac consectetur ac, vestibulum at eros. Nullam quis risus eget urna mollis ornare vel eu leo.

## Images

Quisque consequat sapien eget quam rhoncus, sit amet laoreet diam tempus. Aliquam aliquam metus erat, a pulvinar turpis suscipit at.

![placeholder](https://placehold.it/800x400 "Large example image")
![placeholder](https://placehold.it/400x200 "Medium example image")
![placeholder](https://placehold.it/200x200 "Small example image")

## Tables

Aenean lacinia bibendum nulla sed consectetur. Lorem ipsum dolor sit amet, consectetur adipiscing elit.

<table>
  <thead>
    <tr>
      <th>Name</th>
      <th>Upvotes</th>
      <th>Downvotes</th>
    </tr>
  </thead>
  <tfoot>
    <tr>
      <td>Totals</td>
      <td>21</td>
      <td>23</td>
    </tr>
  </tfoot>
  <tbody>
    <tr>
      <td>Alice</td>
      <td>10</td>
      <td>11</td>
    </tr>
    <tr>
      <td>Bob</td>
      <td>4</td>
      <td>3</td>
    </tr>
    <tr>
      <td>Charlie</td>
      <td>7</td>
      <td>9</td>
    </tr>
  </tbody>
</table>

Nullam id dolor id nibh ultricies vehicula ut id elit. Sed posuere consectetur est at lobortis. Nullam quis risus eget urna mollis ornare vel eu leo.










