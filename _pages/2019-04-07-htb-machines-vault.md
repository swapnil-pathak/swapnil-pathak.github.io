---
layout: single
classes: wide
title:  "Walkthrough - Vault"
date: 2019-03-24
categories:
    - "hackthebox"
    - walkthrough
tags:
    - machines
    - medium
    - linux
---

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/vault/banner.JPG)

```bash
# Nmap 7.70 scan initiated Wed Feb 13 19:37:23 2019 as: nmap -v -p- -sC -sV -oA nmap 10.10.10.109
Nmap scan report for 10.10.10.109
Host is up (0.26s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 a6:9d:0f:7d:73:75:bb:a8:94:0a:b7:e3:fe:1f:24:f4 (RSA)
|   256 2c:7c:34:eb:3a:eb:04:03:ac:48:28:54:09:74:3d:27 (ECDSA)
|_  256 98:42:5f:ad:87:22:92:6d:72:e6:66:6c:82:c1:09:83 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Feb 13 20:04:19 2019 -- 1 IP address (1 host up) scanned in 1615.96 seconds
```
