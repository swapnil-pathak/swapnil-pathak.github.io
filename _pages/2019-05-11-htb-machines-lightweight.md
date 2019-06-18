---
layout: single
classes: wide
title:  "Walkthrough - Lightweight"
date: 2019-05-11
categories:
    - "hackthebox"
    - walkthrough
tags:
    - machines
    - medium
    - linux
---


![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/lightweight/banner.PNG)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

## Enumeration

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```bash
root@kali:~/Lightweight# nmap -v -p- -sC -sV -oA nmap 10.10.10.119
# Nmap 7.70 scan initiated Wed Feb 13 19:33:06 2019 as: nmap -v -p- -sC -sV -oA nmap 10.10.10.119
Nmap scan report for 10.10.10.119
Host is up (0.22s latency).
Not shown: 65532 filtered ports
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 7.4 (protocol 2.0)
| ssh-hostkey:
|   2048 19:97:59:9a:15:fd:d2:ac:bd:84:73:c4:29:e9:2b:73 (RSA)
|   256 88:58:a1:cf:38:cd:2e:15:1d:2c:7f:72:06:a3:57:67 (ECDSA)
|_  256 31:6c:c1:eb:3b:28:0f:ad:d5:79:72:8f:f5:b5:49:db (ED25519)
80/tcp  open  http    Apache httpd 2.4.6 ((CentOS) OpenSSL/1.0.2k-fips mod_fcgid/2.3.9 PHP/5.4.16)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.6 (CentOS) OpenSSL/1.0.2k-fips mod_fcgid/2.3.9 PHP/5.4.16
|_http-title: Lightweight slider evaluation page - slendr
389/tcp open  ldap    OpenLDAP 2.2.X - 2.3.X
| ssl-cert: Subject: commonName=lightweight.htb
| Subject Alternative Name: DNS:lightweight.htb, DNS:localhost, DNS:localhost.localdomain
| Issuer: commonName=lightweight.htb
| Public Key type: rsa
| Public Key bits: 1024
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2018-06-09T13:32:51
| Not valid after:  2019-06-09T13:32:51
| MD5:   0e61 1374 e591 83bd fd4a ee1a f448 547c
|_SHA-1: 8e10 be17 d435 e99d 3f93 9f40 c5d9 433c 47dd 532f
|_ssl-date: TLS randomness does not represent time

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Wed Feb 13 19:49:22 2019 -- 1 IP address (1 host up) scanned in 975.52 seconds
```

Finally, we can get our hands dirty with LDAP which is running on port 389. Lets try to enumerate that using a tool `ldapsearch` on kali. Then after that we will look into port 80 (httpd).

## LDAP search

```bash
root@kali:~/Lightweight# ldapsearch -x -h lightweight.htb -b "dc=lightweight,dc=htb"
# extended LDIF
#
# LDAPv3
# base <dc=lightweight,dc=htb> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# lightweight.htb
dn: dc=lightweight,dc=htb
objectClass: top
objectClass: dcObject
objectClass: organization
o: lightweight htb
dc: lightweight

# Manager, lightweight.htb
dn: cn=Manager,dc=lightweight,dc=htb
objectClass: organizationalRole
cn: Manager
description: Directory Manager

# People, lightweight.htb
dn: ou=People,dc=lightweight,dc=htb
objectClass: organizationalUnit
ou: People

# Group, lightweight.htb
dn: ou=Group,dc=lightweight,dc=htb
objectClass: organizationalUnit
ou: Group

# ldapuser1, People, lightweight.htb
dn: uid=ldapuser1,ou=People,dc=lightweight,dc=htb
uid: ldapuser1
cn: ldapuser1
sn: ldapuser1
mail: ldapuser1@lightweight.htb
objectClass: person
objectClass: organizationalPerson
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: top
objectClass: shadowAccount
userPassword:: e2NyeXB0fSQ2JDNxeDBTRDl4JFE5eTFseVFhRktweHFrR3FLQWpMT1dkMzNOd2R
 oai5sNE16Vjd2VG5ma0UvZy9aLzdONVpiZEVRV2Z1cDJsU2RBU0ltSHRRRmg2ek1vNDFaQS4vNDQv
 shadowLastChange: 17691
 shadowMin: 0
 shadowMax: 99999
 shadowWarning: 7
 loginShell: /bin/bash
 uidNumber: 1000
 gidNumber: 1000
 homeDirectory: /home/ldapuser1

 # ldapuser2, People, lightweight.htb
 dn: uid=ldapuser2,ou=People,dc=lightweight,dc=htb
 uid: ldapuser2
 cn: ldapuser2
 sn: ldapuser2
 mail: ldapuser2@lightweight.htb
 objectClass: person
 objectClass: organizationalPerson
 objectClass: inetOrgPerson
 objectClass: posixAccount
 objectClass: top
 objectClass: shadowAccount
 userPassword:: e2NyeXB0fSQ2JHhKeFBqVDBNJDFtOGtNMDBDSllDQWd6VDRxejhUUXd5R0ZRdms
  zYm9heW11QW1NWkNPZm0zT0E3T0t1bkxaWmxxeXRVcDJkdW41MDlPQkUyeHdYL1FFZmpkUlF6Z24x
 shadowLastChange: 17691
 shadowMin: 0
 shadowMax: 99999
 shadowWarning: 7
 loginShell: /bin/bash
 uidNumber: 1001
 gidNumber: 1001
 homeDirectory: /home/ldapuser2

 # ldapuser1, Group, lightweight.htb
 dn: cn=ldapuser1,ou=Group,dc=lightweight,dc=htb
 objectClass: posixGroup
 objectClass: top
 cn: ldapuser1
 userPassword:: e2NyeXB0fXg=
 gidNumber: 1000

 # ldapuser2, Group, lightweight.htb
 dn: cn=ldapuser2,ou=Group,dc=lightweight,dc=htb
 objectClass: posixGroup
 objectClass: top
 cn: ldapuser2
 userPassword:: e2NyeXB0fXg=
 gidNumber: 1001

 # search result
 search: 2
 result: 0 Success

# numResponses: 9
# numEntries: 8
```

Looks like there are two users here `ldapuser1` and `ldapuser2` in the `people` OU. We also have password hashes (base64 encoded) for both of them.

```yaml
B64decoded: {crypt}$6$3qx0SD9x$Q9y1lyQaFKpxqkGqKAjLOWd33Nwdhj.l4MzV7vTnfkE/g/Z/7N5ZbdEQWfup2lSdASImHtQFh6zMo41ZA./44/
```

The base64 decoded value shows us that the password is `{crypt}` hashed and `$6$` means six iterations and this `3qx0SD9x` is the salt. The passwords are virtually unbreakable. Well that was it from `ldapsearch`. Let's move to port 80.

## HTTP service

Looking at the website and kind of just clicking on tabs shows us this.

![port80-index]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/lightweight/index.PNG)

![port80-info]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/lightweight/info.PNG)

Looks like we cannot use and directory listing tools with this box.

![port80-stats]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/lightweight/stats.PNG)

The list of banned IPs does not show a list (I was expecting a list!).

![port80-user]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/lightweight/user.PNG)

There we have it, ssh access right off the bat. So lets login using our IP as the username and password.

```bash
root@noone:/home/pswapnil/Retired/Lightweight# ssh 10.10.**.**@lightweight.htb
The authenticity of host 'lightweight.htb (10.10.10.119)' can't be established.
ECDSA key fingerprint is SHA256:FWyyew+o9WoPYkfIKGEbTMsexks1z8ZkSUs9O+2AMSU.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'lightweight.htb,10.10.10.119' (ECDSA) to the list of known hosts.
10.10.**.**@lightweight.htb's password:
[10.10.**.**@lightweight ~]$ ls -la
total 12
drwx------.  4 10.10.**.** 10.10.**.**  91 May 15 16:51 .
drwxr-xr-x. 14 root         root         234 May 15 16:50 ..
-rw-r--r--.  1 10.10.**.** 10.10.**.**  18 Apr 11  2018 .bash_logout
-rw-r--r--.  1 10.10.**.** 10.10.**.** 193 Apr 11  2018 .bash_profile
-rw-r--r--.  1 10.10.**.** 10.10.**.** 246 Jun 15  2018 .bashrc
drwxrwxr-x.  3 10.10.**.** 10.10.**.**  18 May 15 16:51 .cache
drwxrwxr-x.  3 10.10.**.** 10.10.**.**  18 May 15 16:51 .config
[10.10.**.**@lightweight ~]$ cd ..
[10.10.**.**@lightweight home]$ ls -la
total 0
drwxr-xr-x. 14 root         root         234 May 15 16:50 .
dr-xr-xr-x. 17 root         root         224 Jun 13  2018 ..
drwx------.  2 10.10.14.104 10.10.14.104  62 May 15 10:44 10.10.14.104
drwx------.  2 10.10.14.176 10.10.14.176  62 May 15 14:49 10.10.14.176
drwx------.  4 10.10.14.2   10.10.14.2    91 Nov 16 22:39 10.10.14.2
drwx------.  2 10.10.14.201 10.10.14.201  62 May 15 13:58 10.10.14.201
drwx------.  2 10.10.14.224 10.10.14.224  62 May 15 16:41 10.10.14.224
drwx------.  4 10.10.14.231 10.10.14.231  91 May 15 16:50 10.10.14.231
drwx------.  2 10.10.14.96  10.10.14.96   62 May 15 10:43 10.10.14.96
drwx------.  2 10.10.15.132 10.10.15.132  62 May 15 07:14 10.10.15.132
drwx------.  4 10.10.**.** 10.10.**.**  91 May 15 16:51 10.10.**.**
drwx------.  2 127.0.0.1    127.0.0.1     62 May 15 10:14 127.0.0.1
drwx------.  4 ldapuser1    ldapuser1    181 Jun 15  2018 ldapuser1
drwx------.  4 ldapuser2    ldapuser2    197 Jun 21  2018 ldapuser2
```

I cannot get into any other directories. So its `LinEnum` time. You can find the link to the repository [here](https://github.com/rebootuser/LinEnum).

```
[+] Files with POSIX capabilities set:
/usr/bin/ping = cap_net_admin,cap_net_raw+p
/usr/sbin/mtr = cap_net_raw+ep
/usr/sbin/suexec = cap_setgid,cap_setuid+ep
/usr/sbin/arping = cap_net_raw+p
/usr/sbin/clockdiff = cap_net_raw+p
/usr/sbin/tcpdump = cap_net_admin,cap_net_raw+ep
```

So this is the only interesting information we got from LinEnum. Looks like there's tcpdump 
