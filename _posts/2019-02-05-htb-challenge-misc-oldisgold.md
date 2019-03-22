---
layout: single
classes: wide
title:  "0ld is g0ld"
date: 2019-02-05
categories:
    - "hackthebox"
tags:
    - challenges
    - misc
excerpt:  "This particular challenge is a good starter to your journey as a challenge solver! Take a moment to appreciate the beauty of old algorithms, without them we would not be able to build cyber security so much. This challenge will earn you 10 points which is not a lot but you got to start somewhere."
---
This particular challenge is a good starter to your journey as a challenge solver! Take a moment to appreciate the beauty of "old" algorithms, without them we would not be able to build cyber security so much. This challenge will earn you 10 points which is not a lot but you got to start somewhere.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Challenges-Misc/morse/banner.PNG)

## Spies everywhere!

Before following this walkthrough, I highly recommend trying to get the flag yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

Follow this link and download the file under 0ld is g0ld section as shown below. You will have to login in order to do that.

```console
htb@noone:~/misc/old_is_gold$ unzip 0ld_is_g0ld.zip
Archive:  0ld_is_g0ld.zip
  [0ld_is_g0ld.zip] 0ld is g0ld.pdf password: hackthebox
inflating: 0ld is g0ld.pdf
htb@noone:~/misc/old_is_gold$ ls
'0ld is g0ld.pdf'  0ld_is_g0ld.zip
```

Well, so we have a pdf file. Let's try to open it.

Turns out its password protected. We will have to see if we can crack its password to see what's in there. I am going to use [pdfcrack](https://www.cyberciti.biz/tips/linux-howto-crack-recover-pdf-file-password.html) to do that.

You might want to lookup the options to use it if you haven't already. Use `pdfcrack -h` to see the options and select what suits your need the best.

I am going to do a dictionary attack, for that I am going to use [rockyou.txt](https://wiki.skullsecurity.org/Passwords) file. You can try with other wordlists just for trying.

````console
htb@noone:~/misc/old_is_gold$ pdfcrack -w /usr/share/wordlists/rockyou.txt -f "0ld is g0ld.pdf"
PDF version 1.6
Security Handler: Standard
V: 2
R: 3
P: -1060
Length: 128
Encrypted Metadata: True
FileID: 5c8f37d2a45eb64e9dbbf71ca3e86861
U: 9cba5cfb1c536f1384bba7458aae3f8100000000000000000000000000000000
O: 702cc7ced92b595274b7918dcb6dc74bedef6ef851b4b4b5b8c88732ba4dac0c
Average Speed: 48468.9 w/s. Current Word: 'dichan'
```snip```
found user-password: 'jumanji69'
````

We will use this password to open the pdf.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Challenges-Misc/morse/pdfout.png)

Woah! Take a bow [Samuel Morse](https://en.wikipedia.org/wiki/Samuel_Morse) (I didn't know it was him. A little search helped me). But we are here for the flag. What does this picture mean? If we scroll down and zoom in, we see something.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Challenges-Misc/morse/morse.png)

Look at that! We have a morse code to crack. I will use an online translator to do this (Maybe I will create a simple tool).

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Challenges-Misc/morse/morsetranslate.png)

Voila! We have the FLAG and we can use this to gain out points on HackTheBox. Don't forget to enclose the flag in *HTB{}* because that's the format.

Happy Hacking! Cheers!
