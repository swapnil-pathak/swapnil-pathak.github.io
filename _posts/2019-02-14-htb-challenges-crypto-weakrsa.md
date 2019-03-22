---
layout: single
classes: wide
title:  "Weak RSA"
date: 2019-02-14
categories:
    - "hackthebox"
tags:
    - challenges
    - crypto
excerpt:  "Breaking the infamous RSA algorithm. It has been the gold standard for public-key cryptography. There's a catch though, if you implement it badly, your ciphertext is no longer safe. Given a few minutes and a bit of RSA knowledge should do the trick for this challenge. Read here for more information on this."
---
Breaking the infamous RSA algorithm. It has been the gold standard for public-key cryptography. There's a catch though, if you implement it badly, your ciphertext is no longer safe. Given a few minutes and a bit of RSA knowledge should do the trick for this challenge. Read here for more information on this.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Challenges-Crypto/weakrsa/banner.PNG)

## Your key is Public

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

Follow this [link](https://www.hackthebox.eu/home/challenges/Crypto) and download the file under Weak RSA section. You will have to login in order to do that.

```console
htb@noone:~/crypto/weakrsa$ unzip weak-rsa.zip
Archive:  weak-rsa.zip
[weak-rsa.zip] flag.enc password:
  inflating: flag.enc
  inflating: key.pub
htb@noone:~/crypto/weakrsa$
```

Well, so we have a two files. Let's try to open them.

```console
htb@noone:~/crypto/weakrsa$ cat key.pub
-----BEGIN PUBLIC KEY-----
MIIBHzANBgkqhkiG9w0BAQEFAAOCAQwAMIIBBwKBgQMwO3kPsUnaNAbUlaubn7ip
4pNEXjvUOxjvLwUhtybr6Ng4undLtSQPCPf7ygoUKh1KYeqXMpTmhKjRos3xioTy
23CZuOl3WIsLiRKSVYyqBc9d8rxjNMXuUIOiNO38ealcR4p44zfHI66INPuKmTG3
RQP/6p5hv1PYcWmErEeDewKBgGEXxgRIsTlFGrW2C2JXoSvakMCWD60eAH0W2PpD
qlqqOFD8JA5UFK0roQkOjhLWSVu8c6DLpWJQQlXHPqP702qIg/gx2o0bm4EzrCEJ
4gYo6Ax+U7q6TOWhQpiBHnC0ojE8kUoqMhfALpUaruTJ6zmj8IA1e1M6bMqVF8sr
lb/N
-----END PUBLIC KEY-----
htb@noone:~/crypto/weakrsa$cat flag.enc
_vc[~kZ1Ĩ4I9V^G(+3Lu"$F0VP־j@|j{¾,YEXx,cN&Hl2Ӎ[o
```

Looks like we have a public key which was used to produce the encrypted gibberish. Let's learn more about [RSA](https://www.geeksforgeeks.org/rsa-algorithm-cryptography/).

We can derive from the reading that if the `p` and `q` values are smaller primes, we can break the RSA algorithm! That might be the case in this challenge (I hope!).

A little Google Fu got me this tool. We can leverage it to try and decrypt the `flag.enc` file contents.

```console
htb@noone:~/crypto/weakrsa$ python RsaCtfTool/RsaCtfTool.py --publickey key.pub --uncipherfile flag.enc
[+] Clear text : !ϲo@gX{Dn(Dx".,-)!6WQ+L~J9߬{_SDYCߚ4LU
   p/AHTB{s1mpl3*********4tt4ck}
htb@noone:~/crypto/weakrsa$
```

Voila! We have the FLAG and we can use this to gain out points on HackTheBox. Just copy paste it on the HackTheBox portal.

Happy Hacking! Cheers!
