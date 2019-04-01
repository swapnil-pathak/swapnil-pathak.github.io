---
layout: single
classes: wide
title:  "Walkthrough - Carrier"
date: 2019-03-23
categories:
    - "hackthebox"
tags:
    - machines
    - medium
    - linux
---

This machine was fun to do.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/banner.JPG)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```bash
root@noone:/home/pswapnil/Carrier# nmap -v -p- -sC -sV -oA carrier 10.10.10.105
# Nmap 7.70 scan initiated Mon Feb 11 15:32:30 2019 as: nmap -v -p- -sC -sV -oA carrier 10.10.10.105
Increasing send delay for 10.10.10.105 from 0 to 5 due to 42 out of 139 dropped probes since last increase.
Nmap scan report for 10.10.10.105
Host is up (0.26s latency).
Not shown: 65532 closed ports
PORT   STATE    SERVICE VERSION
21/tcp filtered ftp
22/tcp open     ssh     OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 15:a4:28:77:ee:13:07:06:34:09:86:fd:6f:cc:4c:e2 (RSA)
|   256 37:be:de:07:0f:10:bb:2b:b5:85:f7:9d:92:5e:83:25 (ECDSA)
|_  256 89:5a:ee:1c:22:02:d2:13:40:f2:45:2e:70:45:b0:c4 (ED25519)
80/tcp open     http    Apache httpd 2.4.18 ((Ubuntu))
| http-cookie-flags:
|   /:
|     PHPSESSID:
|_      httponly flag not set
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Login
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Feb 11 15:46:38 2019 -- 1 IP address (1 host up) scanned in 847.77 seconds
```

So we have a filtered FTP port, port 22 running SSH and port 80 running Apache httpd server.
The SSH and FTP probably won't get us anywhere. So let's start with HTTP.

We get this when we visit the webpage.

![webpage]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/webpage.png)

We can see two error codes. Generally displayed to show some errors in the underlying system configuration. Let's try to get all the available directories on the webpage.

```bash
root@noone:/home/pswapnil/Carrier# wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt --hc 404 http://10.10.10.105/FUZZ                                                                                        

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz documentation for more information.                                                         

********************************************************
* Wfuzz 2.2.11 - The Web Fuzzer                        *
********************************************************

Target: http://10.10.10.105/FUZZ
Total requests: 950

==================================================================
ID      Response   Lines      Word         Chars          Payload
==================================================================

000244:  C=301      9 L       28 W          310 Ch        "css"
000262:  C=301      9 L       28 W          312 Ch        "debug"
000294:  C=301      9 L       28 W          310 Ch        "doc"
000430:  C=301      9 L       28 W          310 Ch        "img"
000470:  C=301      9 L       28 W          309 Ch        "js"
000844:  C=301      9 L       28 W          312 Ch        "tools"

Total time: 24.39516
Processed Requests: 950
Filtered Requests: 944
Requests/sec.: 38.94214

```

Great. So we found 6 directories. Let's visit each one and go from there.


```bash
root@noone:/home/pswapnil/Carrier# snmpwalk -v2c -c public 10.10.10.105 .1
Created directory: /var/lib/snmp/mib_indexes
iso.3.6.1.2.1.47.1.1.1.1.11 = STRING: "SN#NET_45JDX23"
iso.3.6.1.2.1.47.1.1.1.1.11 = No more variables left in this MIB View (It is past the end of the MIB tree)
```
