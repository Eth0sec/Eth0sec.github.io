---
title: PicoCTF 2024 Web Exploitation Write-Ups
excerpt: "This post contains writeups for all the web exploitation challenges I was able to solve for PicoCTF this year."
categories:
  - ctf
  - web-security
tags:
  - picoctf
header:
  teaser: /assets/images/teasers/picoctf.jpg
---

Hi all! I wanted to document all the web challenges I was able to solve for PicoCTF 2024 which took place March 12th-26th this year. I’m still quite new to CTFs, but found each of these challenges to be quite beginner friendly. I was able to solve five of the seven web challenges.

## WebDecode

This challenge is meant to teach the user about how to inspect HTML and edit frontend source code using inspector. This one was quite straightforward.

I was able to find a suspicious looking base64 encoded string in the `'notify_true'` attribute on the about page.

![Base 64 Hidden String](/assets/images/posts/ctf/picoctf2024-writeup/webdecode-source.jpg)

I used `Hurl` on my linux terminal to decode the string, and there is the flag!

![Hurl Decode](/assets/images/posts/ctf/picoctf2024-writeup/webdecode-hurl.jpg)

**Flag:** *picoCTF{web_succ3ssfully_d3c0ded_1f832615}*

## Bookmarklet

This one was pretty trivial as well. It’s meant to teach users about bookmarklets which are small javascript programs that stored as the URL of a bookmark in your browser. When you click on the bookmarklet, it executes the JavaScript code and performs some action on the current web page you’re viewing. They are a way to add functionality to your browser without having to install a full-fledged extension or add-on.

The challenge URL gives you the javascript code you need to solve the challenge. I simply pasted into the console and got the flag.

![Bookmarklets Home Page With Code](/assets/images/posts/ctf/picoctf2024-writeup/bookmarklet-javascript.jpg)

![Bookmarklet Pasted to Console](/assets/images/posts/ctf/picoctf2024-writeup/bookmarklet-console.jpg)

**Flag:** *picoCTF{p@g3_turn3r_6bbf8953}*

## IntroToBurp

The challenge hint for this one urges you to use BurpSuite, so off I went. The instance link takes you to a simple web page with a registration form. I tried fuzzing the web form a bit, and it became clear that it would accept any of my inputs.

![Web Registration Form](/assets/images/posts/ctf/picoctf2024-writeup/introtoburp-registration.jpg)

This brought me to a 2FA page. I put in a random value for the OTP to see what would happen and intercepted the request. I got back nothing more than an `'invalid OTP'` response when forwarding it on. I then tried to see if I could manipulate the request in any possible way including changing the request method or headers, but was not so lucky there. After a while of fiddling around, I tried replacing the content-type to `application/xml` to see if matching a type from the accept header would do anything, lo and behold, the key revealed itself to me!

![BurpSuite Intercepted Request](/assets/images/posts/ctf/picoctf2024-writeup/introtoburp-request.jpg)

![IntroToBurp Key](/assets/images/posts/ctf/picoctf2024-writeup/introtoburp-key.jpg)

**Flag:** *picoCTF{#OTP_Bypvss_SuCc3$S_b3fa4fla}*

## Unminify

This challenge presents the player with a simple flag distribution website. We are notified that the browser has successfully received the flag.

![Unminify Page](/assets/images/posts/ctf/picoctf2024-writeup/unminify-page.jpg)

This challenge is meant to teach players how minifying web source code works and what it looks like. It’s a neat concept, but…it doesn’t make for a very compelling CTF challenge. I was able to hit `ctrl-u` to view the source code, wrap the text, and find the flag with ease. Not too sure why this one earns 100 pts, it’s far too simple.

![Unminify Source Code](/assets/images/posts/ctf/picoctf2024-writeup/unminify-source.jpg)

**Flag:** *picoCTF{pr3tty_c0d3_51d374f0}*

## Trickster

This was the first challenge of the CTF that gave me a little bit of push back. The link provided takes the player to a PNG processing application. Looks like we have the ability to browse and upload our own PNG files.

First things first, I wanted to get a better sense of the file structure of the site, so I went searching in the `robots.txt` file for any meaningful directories.

![Trickster Robots.txt File](/assets/images/posts/ctf/picoctf2024-writeup/trickster-robots.jpg)

Looks like there's an `uploads` directory that crawlers are forbidden from indexing. There’s also an `'instructions.txt'` file that’s calling my name. Bots might not be able to access it, but maybe I can?

![Trickster Instructions.txt File](/assets/images/posts/ctf/picoctf2024-writeup/trickster-instructions.jpg)

Hmmm, very curious. It looks like files uploaded to this site not only require the `.png` extension type, but also use magic bytes for validation as well. On one hand, this is great because I knew the validation method for images now, but it also wouldn’t be as easy as appending `.png` at the end of any payload.

The instructions text file tell us partially the hex digits for PNG magic bytes. With some quick google searching, I found the full sequence to be `89 50 4E 47 0D 0A 1A 0A`. Huge shout out to `garykessler.net` for providing a longstanding and comprehensive list of file signatures.

I still lacked some crucial information about the backend source code of the site, so I copy and pasted the URL into Wappalyzer and it looks to be running on `PHP 8.0.30`. Now I had everything I needed to craft a payload and hopefully get a shell.

![Trickster Wappalyzer](/assets/images/posts/ctf/picoctf2024-writeup/trickster-wappalyzer.jpg)

I wanted to start with a simple test, so I used the `phpinfo()` function to gather some intel on the configuration of the site. I embedded the function within a PNG file that has magic bytes prepended to it. The output still needs to have a`.php` file extension appended to it in order for any kind of code execution to actually work.

```bash
printf '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A<?php phpinfo(); __halt_compiler(); ?>' > totallynotphp.png.php
```

The test upload proved successful! Trying to access the upload folder is forbidden but you can access the file directly at `uploads/totallynotphp.png.php`. `phpinfo()` did its job and we can see all of the PHP information of the site.

![Trickster Upload Successful](/assets/images/posts/ctf/picoctf2024-writeup/trickster-uploaded.jpg)

![Trickster Upload Folder Forbidden](/assets/images/posts/ctf/picoctf2024-writeup/trickster-forbidden.jpg)

![Trickster PHP Information](/assets/images/posts/ctf/picoctf2024-writeup/trickster-phpinfo.jpg)

Now that I've confirmed this exploit works as suspected, I crafted a reverse shell payload. Here's the code:

```bash
printf '\x89\x50\x4E\x47\x0D\x0A\x1A\x0A<?php if (isset($_GET["cmd"])) { system($_GET["cmd"] . " 2>&1"); } ?>' > phprevshell.png.php
```

After uploading the file, I accessed it. Turns out I wasn't able to freely navigate the file system, but could list the contents of the upload folder using `ls ../`. I was able to find a very suspicious looking text file named `HFQWKODGMIYTO.txt`.

![Trickster Reverse Shell](/assets/images/posts/ctf/picoctf2024-writeup/trickster-revshell.jpg)

I accessed the file and found the flag within! This ended up being a really enjoyable challenge.

![Trickster Flag](/assets/images/posts/ctf/picoctf2024-writeup/trickster-flag.jpg)

**Flag:** *picoCTF{c3rt!fi3d_Xp3rt_tr1ckst3r_9ae8fb17}*