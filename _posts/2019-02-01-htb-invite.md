---
classes: wide
title:  "Walkthrough - HTB Invite code (Hints only)"
date: 2019-02-01
categories:
    - "hackthebox"
    - walkthrough
---
Getting the invite code to login and start hacking!

Unlinke many other CTF-like or Real-world scenario based services, to start your arduous journey with HackTheBox, you will need to obtain an invite code to prove your worth. To get the ball rolling, here is some information on that.

## Try harder!

Before following this walkthrough, I highly recommend trying to get the invite yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

In order to login, you need to find the [invite page](https://www.hackthebox.eu/invite) shown below.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/invite_page.PNG" alt="htb_invite_page">

If you are a "Developer", you would know where to look, right? There are things you could look into what other people might have created (probably the source?).

Other related pages can really be helpful. You could find some information that can be used to gain advantage.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/calm.PNG" alt="calm">

I encourage you to look through what certain functions do. Search on the file names you find. Pretty sure you will get some encoded information. Try to figure out the encoding scheme (might want to check the length of the encoded string - is it a multiple of 4?).

Once you have it, let's try to decode (you can use any online tool available, I prefer commandline) that and see what we can uncover.

Once you get the output, you will know what service request you need to use.

Pretty sure, any output you receive would be encoded. Try to decode by applying the same strategy as before.

Voila! We have the Invite code and we can use this to login to our final destination, HackTheBox.

Happy Hacking! Cheers!
