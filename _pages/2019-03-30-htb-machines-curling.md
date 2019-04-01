---
layout: single
classes: wide
title:  "Walkthrough - Curling"
date: 2019-03-30
categories:
    - "hackthebox"
tags:
    - machines
    - easy
    - linux
---

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/Curling/banner.JPG)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```bash
root@noone:/home/pswapnil/Curling# nmap -v -sC -sV -p- -oA nmap 10.10.10.150
# Nmap 7.70 scan initiated Sat Feb  2 22:09:43 2019 as: nmap -v -sC -sV -p- -oA nmap 10.10.10.150
Nmap scan report for 10.10.10.150
Host is up (0.26s latency).
Not shown: 65521 closed ports
PORT      STATE    SERVICE    VERSION
22/tcp    open     ssh        OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 8a:d1:69:b4:90:20:3e:a7:b6:54:01:eb:68:30:3a:ca (RSA)
|   256 9f:0b:c2:b2:0b:ad:8f:a1:4e:0b:f6:33:79:ef:fb:43 (ECDSA)
|_  256 c1:2a:35:44:30:0c:5b:56:6a:3f:a5:cc:64:66:d9:a9 (ED25519)
80/tcp    open     http       Apache httpd 2.4.29 ((Ubuntu))
|_http-favicon: Unknown favicon MD5: 1194D7D32448E1F90741A97B42AF91FA
|_http-generator: Joomla! - Open Source Content Management
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Home
9001/tcp  filtered tor-orport
9910/tcp  filtered unknown
10693/tcp filtered unknown
12032/tcp filtered unknown
13052/tcp filtered unknown
14121/tcp filtered unknown
33922/tcp filtered unknown
37841/tcp filtered unknown
39026/tcp filtered unknown
52288/tcp filtered unknown
61124/tcp filtered unknown
62918/tcp filtered unknown
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Feb  2 22:55:54 2019 -- 1 IP address (1 host up) scanned in 2770.59 seconds
```

Just two ports open, port 22 for SSH and port 80 for HTTP.
Let's start with HTTP. From the nmap scan we can see that it is a `Joomla` CMS.
