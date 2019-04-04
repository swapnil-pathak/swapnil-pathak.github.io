---
layout: single
classes: wide
title:  "Walkthrough - Frolic"
date: 2019-03-24
categories:
    - "hackthebox"
    - walkthrough
tags:
    - machines
    - easy
    - linux
---

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/banner.JPG)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```bash
root@noone:/home/pswapnil/Curling# nmap -v -p- -sC -sV -oA nmap 10.10.10.111
# Nmap 7.70 scan initiated Thu Feb 14 11:48:48 2019 as: nmap -v -p- -sC -sV -oA nmap 10.10.10.111
Nmap scan report for 10.10.10.111
Host is up (0.25s latency).
Not shown: 65530 closed ports
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 87:7b:91:2a:0f:11:b6:57:1e:cb:9f:77:cf:35:e2:21 (RSA)
|   256 b7:9b:06:dd:c2:5e:28:44:78:41:1e:67:7d:1e:b7:62 (ECDSA)
|_  256 21:cf:16:6d:82:a4:30:c3:c6:9c:d7:38:ba:b5:02:b0 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
1880/tcp open  http        Node.js (Express middleware)
|_http-favicon: Unknown favicon MD5: 818DD6AFD0D0F9433B21774F89665EEA
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Node-RED
9999/tcp open  http        nginx 1.10.3 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD
|_http-server-header: nginx/1.10.3 (Ubuntu)
|_http-title: Welcome to nginx!
Service Info: Host: FROLIC; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -1h49m59s, deviation: 3h10m31s, median: 0s
| nbstat: NetBIOS name: FROLIC, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   FROLIC<00>           Flags: <unique><active>
|   FROLIC<03>           Flags: <unique><active>
|   FROLIC<20>           Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|_  WORKGROUP<1e>        Flags: <group><active>
| smb-os-discovery:
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: frolic
|   NetBIOS computer name: FROLIC\x00
|   Domain name: \x00
|   FQDN: frolic
|_  System time: 2019-02-14T22:41:15+05:30
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2019-02-14 12:11:15
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Feb 14 12:11:21 2019 -- 1 IP address (1 host up) scanned in 1353.08 seconds
```

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/banner.JPG)
