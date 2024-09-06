---
title: UIUCTF 2024 Web & OSINT Writeups
excerpt: "This post contains my personal solutions for the Web and OSINT challenges from UIUCTF"
categories:
  - ctf
  - osint
  - web-security
tags:
  - ssrf-attacks
  - uiuctf
header:
  teaser: /assets/images/teasers/uiuctf.jpg
featured: true
---

I had the chance to participate in UIUCTF 2024 hosted by SigPwny, the cybersecurity club at University of Illinois at Urbana-Champaign. Unfortunately I didn’t have much time to dedicate to this one as I had hoped, but I was still able to solve one of the web challenges and a few of the OSINT challenges. I found them to be pretty enjoyable and log action definitely felt like an appropriate challenge for my skill level!

## Web

### Log-Action

This challenge presents the user with a login page, and the hint provided indicates that the web form is broken. I tried it, and sure enough, it definitely appears so.

![Log Action Broken Form](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-broken.jpg)

We are given a file that has the source code for the site, so that is where I went next to gather some intel.

I was able to determine that the application’s frontend was built using the `Next.js` React framework and the `Tailwind CSS` framework.

In the `docker-compose.yml` setup file, you’re able to see that the flag itself has been mounted inside the container at the path `/usr/share/nginx/html/flag.txt`. Got it, that means I would need to find a way to access the `Nginx` backend service.

![Log Action Docker Compose](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-compose.jpg)

I already knew that Next.js offers server-side rendering (SSR) support. The `packages.json` file revealed the version of Next.js to be `14.1.0`.

![Log Action Packages.Json](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-packages.jpg)

I was able to find a Mitre CVE Record, [CVE-2024-34351](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2024-34351), for an `SSRF` vulnerability affecting Next.js applications version `14.1.0 and lower`. Looks like it has to do with modifying the host header and making sure certain conditions are met.

![Log Action CVE](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-cve-2024-34351.jpg)

The first two conditions are already met, so all I needed to do was find an action that causes a redirect. Luckily, it didn’t take me too long to find something interesting. The `pages.tsx` file for the logout page reveals some redirection functionality. Here’s the JavaScript that is particularly interesting:

```javascript
action={async () => {
  "use server";
  await signOut({ redirect: false });
  redirect("/login");
}}

This action function handles form submission. Server-side code is executed to log a user out without performing an automatic redirection. The user is redirected to the `/login` page after the logout process is complete. Very interesting…server-side actions in Next.js are designed to be executed regardless of the user’s authentication state. This implies that the action function can be triggered without checking if the user is logged in, so let’s try it.

![Log Action Logout Page Code](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-redirect.jpg)

I went to the `/logout` page and was successfully able to log out of the application and be redirected to the login page.

![Log Action Logout Page](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-logout.jpg)

Okay! It’s about time to exploit the application at this point with SSRF. I was actually able to find a really awesome article on this topic, [Digging for SSRF in NextJS apps](https://www.assetnote.io/resources/research/digging-for-ssrf-in-nextjs-apps), that was written only a month before this CTF. Super convenient! Credit to Adam Kues and Shubham Shah for being the first ones to find and document this vulnerability.

I modified a Python `Flask` sample shared in the article to set up a redirection server for my SSRF attack. Here’s the full code:

```python
from flask import Flask, Response, request, redirect

app = Flask(__name__)

@app.route('/', defaults={'path': ''})
@app.route('/<path:path>')
def catch(path):
    if request.method == 'HEAD':
        resp = Response("")
        resp.headers['Content-Type'] = 'text/x-component'
        return resp
    return redirect('http://backend/flag.txt')

if __name__ == "__main__":
    app.run(port=5000, debug=True)
```

The server is configured to run on port 5000. For every `HEAD` request the server receives, a `200` response with the `Content-Type: text/x-component` attribute will be received. This will pass the CORS preflight check. For `GET` requests, `302` will be returned and we’ll get redirected to the response URL to access the flag.

I ran the Flask server and then set up a secure tunnel using `Localhost.run` to make it publicly accessible.

![Log Action Flask Server and Localhost.run](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-flask.jpg)

All that needs to be done now is intercept the request, edit the `host` and `origin` attributes and forward the modified request to the attack server, and the flag should reveal itself in the response body.

![Log Action BurpSuite Modified Request](/assets/images/posts/ctf/uiuctf2024-writeup/logaction-request.jpg)

*Flag:* uiuctf{close_enough_nextjs_server_actions_welcome_back_php}

## OSINT

### Hip With the Youth

This was the first in a three-part series. We are tasked with finding a flag on the Instagram page of the Long Island Subway Authority (LISA). This is the recommended part to start with.

I searched for `Long Island Subway Authority` through Instagram’s search bar and was easily able to find their page. There was really nothing of consequence there except for a couple of posts and a link to a Threads page in the bio.

![Hip With the Youth Instagram Bio](/assets/images/posts/ctf/uiuctf2024-writeup/hwty-instabio.jpg)

I followed the link and found a really suspicious looking thread that mentions including a flag with the post to boost engagement.

![Hip With the Youth Threads Page](/assets/images/posts/ctf/uiuctf2024-writeup/hwty-thread.jpg)

I followed the thread and the flag was right there in the replies. Super simple first OSINT challenge.

![Hip With the Youth Flag](/assets/images/posts/ctf/uiuctf2024-writeup/hwty-flag.jpg)

*flag:* uiuctf{7W1773r_K!113r_321879}

### An Unlikely Partnership

![LISA LinkedIn Page](/assets/images/posts/ctf/uiuctf2024-writeup/aup-linkedin.jpg)

After some snooping around, I noticed there was an endorsement under the `Skills` section of the page.

![LISA LinkedIn Page Skills & Endorsement](/assets/images/posts/ctf/uiuctf2024-writeup/aup-endorsement.jpg)

![LISA LinkedIn Page Skills & Endorsement](/assets/images/posts/ctf/uiuctf2024-writeup/aup-endorsement2.jpg)

I clicked on the profile and it looks like this is our influencer. The flag is right there in the `About` section.

![UIUChan LinkedIn Page & Flag](/assets/images/posts/ctf/uiuctf2024-writeup/aup-flag.jpg)

*flag:* uiuctf{0M160D_U1UCCH4N_15_MY_F4V0r173_129301}

### The Weakest Link

This is the final part of this OSINT challenge series. It tasks us with finding the secret Spotify collaboration they have planned together.

I found a link in the `Contact Info` section of the LinkedIn profile that led me to the Spotify profile with only a single public playlist. This actually wound up being a bit more difficult than I had originally thought it would be.

![LinkedIn Contact Info Link](/assets/images/posts/ctf/uiuctf2024-writeup/twl-contactinfo.jpg)

I checked each of the artist profiles in the playlist to see if any had been made for the challenge, but they all looked legit. I also checked the followers list, but it just appeared to be other participants of this CTF and none that were created solely for the challenge.

![Spotify Profile](/assets/images/posts/ctf/uiuctf2024-writeup/twl-profile.jpg)

Seeing all the other participants who followed the account, it dawned on me. You can get information on which songs people you follow are currently listening to *including* which playlist those songs belong to, if any.

What do ya know, they were listening to a song on a playlist named `songs for train lovers`. A very peculiar name…isn’t it. Perhaps this is the collaborative playlist with LISA?

![Spotify Profile](/assets/images/posts/ctf/uiuctf2024-writeup/twl-activity.jpg)

I entered the name of the playlist into the search bar and filtered by `playlists`, accessed the playlist. The flag was located in the description of the playlist. Quite a fun series of OSINT challenges!

![Spotify Profile](/assets/images/posts/ctf/uiuctf2024-writeup/twl-playlist.jpg)

*flag:* uiuctf{7rU1Y_50N65_0F_7H3_5UMM3r_432013}