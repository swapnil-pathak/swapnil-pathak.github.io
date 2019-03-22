---
classes: wide
title:  "HTB Invite code"
date: 2019-02-01
categories:
    - "hackthebox"
excerpt:  "Unlinke many other CTF-like or Real-world scenario based services, to start your arduous journey with HackTheBox, you will need to obtain an invite code to prove your worth. To get the ball rolling, here is some information on that."
---
Getting the invite code to login and start hacking!

Unlinke many other CTF-like or Real-world scenario based services, to start your arduous journey with HackTheBox, you will need to obtain an invite code to prove your worth. To get the ball rolling, here is some information on that.

## Try harder!

Before following this walkthrough, I highly recommend trying to get the invite yourself! Just like you will hear from everyone else, try harder! (if you cannot find it)

In order to login, you need to find the [invite page](https://www.hackthebox.eu/invite) shown below.

<img src="{{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/invite_page.PNG" alt="htb_invite_page">

Once you find it, go to the Developer console (Ctrl + Shift + I), if you look closely you will see a JavaScript link to the `*inviteapi*`.

Try pasting the specified [URL](https://www.hackthebox.eu/js/inviteapi.min.js) in the search bar and let's see what we find out.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/make_invite_code.PNG)

Guess what! There is an API that actually generates an invite code for you. Pretty cool isn't it? Let's use the console to fire up this API (`makeInviteCode()`). Well the [return code](https://www.restapitutorial.com/httpstatuscodes.html) is 200 and we receive a string with [base64 encoding](https://en.wikipedia.org/wiki/Base64).

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/base64_invite.PNG)

Let's try to decode (you can use any online tool available, I prefer commandline) that and see what we can uncover.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/base64_decode.PNG)

Great! So it gave us an extended URL to send a [POST request](https://en.wikipedia.org/wiki/POST_(HTTP)) to. Let's do that. I will use [CURL](https://curl.haxx.se/docs/httpscripting.html) because like I said I like commandline. `curl -XPOST https://www.hackthebox.eu/api/invite/generate` should do the trick.

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/base64_invitecode.PNG)

Success! We got the invite code but as you can see its encoded (PS: I have hidden some of it just in case.). Let's try using Base64 decoding again. Let's do `echo "THE CODE HERE" | base64 -d` and we get ...

![alt]({{ site.url }}{{ site.baseurl }}/assets/images/HTB_images/Invite/invitecode.PNG)

Voila! We have the Invite code and we can use this to login to our final destination, HackTheBox.

Happy Hacking! Cheers!
