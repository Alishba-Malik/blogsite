---
title: TheCyberThesisCtf F-24 - Web - Cookie
date: '2024-09-29'
tags: ['Web', 'ctf', 'Cookie','Writeup']
draft: false
summary: Manipulate the cookie to get the admin access.
---

Hey, Back again!!!

But this time with something a bit more fun, I was the author of this web challenge in **thecyberthesisctf** specially organized for our f-24 batch. 
This challenge was all about *cookies* — no, not the kind you eat, but the kind stored in your browser!

Here’s the sneak peek: you log in with whatever username and password you want (seriously, it doesn’t matter), but once you hit submit, boom: **Access Denied, you are not the admin.**

Seems like a dead end, right? Not at all. The real trick here is manipulating the cookies stored in your browser. 

yass kinda fun na, so let’s get into it.

Here is the challenge:

![Screenshot](/static/writeups/thecyberthesisctf/f-24/web/cookie/1.png)

You can access the source files [here](https://github.com/Alishba-Malik/Cookie).

### Solution:

You land on the login page. Enter whatever you want for the username and password — **anything** works. When you hit "Login," you get redirected to a dashboard, but guess what? You’re not the admin, and the site kindly tells you: **Access Denied**. You might think that’s the end of the road, but we know better.

![Screenshot](/static/writeups/thecyberthesisctf/f-24/web/cookie/2.png)

![Screenshot](/static/writeups/thecyberthesisctf/f-24/web/cookie/3.png)

Time to get into the real fun. Open up your browser’s **Developer Tools** (usually F12 or right-click → Inspect), then navigate to the **Application** tab.
Look under the **Cookies** section. A little digging reveals something interesting: a cookie named `is_Admin`, and its value is currently set to `'0'`. Hmm, that looks suspiciously like the reason we’re being denied access.

![Screenshot](/static/writeups/thecyberthesisctf/f-24/web/cookie/4.png)

Here’s where the magic happens: click on the `is_Admin` cookie and change its value from `'0'` to `'1'`.

![Screenshot](/static/writeups/thecyberthesisctf/f-24/web/cookie/5.png)

Once you’ve changed the cookie value, simply refresh the page. **Boom!** You’ve now tricked the website into thinking you’re an admin, and the flag is yours:

![Screenshot](/static/writeups/thecyberthesisctf/f-24/web/cookie/6.png)

### Flag:

```powershell
TCT{U_kN0w_wH47_4_c00k13_15}
```

So yeah, the lesson here? Never underestimate what’s happening behind the scenes with cookies, especially when they’re left vulnerable like this. It’s a perfect reminder that not all security flaws need to be complicated — sometimes it’s the easy stuff that catches you off guard. The best part is, you not only get the flag, but also a little chuckle knowing you beat the system with a single cookie tweak. Not bad for a day’s work, right?

**Happy Hacking!!!**