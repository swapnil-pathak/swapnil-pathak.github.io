---
layout: single
classes: wide
title:  "You can do it!"
date: 2019-02-14
categories:
    - "hackthebox"
tags:
    - challenges
    - crypto
excerpt:  "Cryptography is an art of hiding data in plain sight. This challenge presents you with the easiest way to obfuscate your data. Visit this section to learn more."
---
Cryptography is an art of hiding data in plain sight. This challenge presents you with the easiest way to obfuscate your data. Visit this section to learn more.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Challenges-Crypto/you_can_doit/banner.PNG)

## Observe!

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

Follow this [link](https://www.hackthebox.eu/home/challenges/Crypto) and download the file under You can do it! section. You will have to login in order to do that.

```console
htb@noone:~/crypto/you_can_do_it$ unzip you_can_do_it.zip
Archive:  you_can_do_it.zip
	[you_can_do_it.zip] you_can_do_it.txt password:
extracting: you_can_do_it.txt
htb@noone:~/crypto/you_can_do_it$ cat you_can_do_it.txt
YHAOANUTDSYOEOIEUTTC!
htb@noone:~/crypto/you_can_do_it$
```

Well, so we have a txt file with some rearranged text it seems.

If you OBSERVE closely, there is an "!" mark at the end of the string. Also, it looks like the letters are just rearranged with a specific interval. Just by looking I could make out words like "YOU", "CAN" and others. If you count the number of characters it's 21 and there looks like a gap of 2 letters for obtaining the plaintext. It's a [Caesar Box](https://www.wikihow.com/Decode-a-Caesar-Box-Code).

I will use this [website](https://www.dcode.fr/caesar-box-cipher) to crack the code!

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Challenges-Crypto/you_can_doit/flag.PNG)

Voila! We have the FLAG and we can use this to gain out points on HackTheBox. Don't forget to enclose the flag in *HTB{}* because that's the format.

Happy Hacking! Cheers!
