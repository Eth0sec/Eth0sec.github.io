---
title: Introducing the Active Directory Ethical Hacking Series
excerpt: "Dive into my new blog series on Active Directory hacking, where I’ll guide you through essential tactics and procedures leveraged against Active Directory. Later, learn how these critical vulnerabilities work so that you can fortify your Active Directory defenses."
categories:
  - active-directory
  - penetration-testing
tags:
  - credential-dumping
  - enumeration
  - exploitation
  - lateral-movement
  - persistence
  - privilege-escalation
header:
  teaser: /assets/images/teasers/ad-hacking.jpg
---

# Introducing the Active Directory Ethical Hacking Series

It’s official, identity-driven attacks have now become the top cyber threats targeting enterprise organizations according to [IBM’s X-Force Threat Intelligence Index 2024](https://www.ibm.com/reports/threat-intelligence). With data gathered from billions of event logs, IBM has identified some worrying trends. There has been a roughly 71% percent increase in identity attacks from the previous year alone. Other security firms, such as [Cloudstrike](https://www.crowdstrike.com/global-threat-report/), have reported similar outcomes.

Active Directory is a blazing hot target for identity-driven attacks such as adversary-in-the-middle or kerberoasting. It’s the most popular directory service for identity storage with an overwhelming [90% adoption rate among Fortune 1000 companies](https://www.crowdstrike.com/blog/attackers-set-sights-on-active-directory-understanding-your-identity-exposure/). Many companies end up falling short in terms of a proper Active Directory security posture. Semperis, a company that specializes in Active Directory security, reports that [88% of Microsoft clients experience attacks against AD due to misconfigurations](https://www.crowdstrike.com/blog/attackers-set-sights-on-active-directory-understanding-your-identity-exposure/).

Preventing these attacks ensures critical systems, applications, and confidential information remains intact and that business continuity is uninterrupted. Ignoring Active Directory isn’t an option if you’re a cybersecurity professional. With that being said, I’d like to introduce a new blog series of mine aimed at hacking Active Directory.

Each post will cover the critical phases of active directory penetration testing from initial enumeration to remediation. Some of the techniques, tactics, and procedures covered in this blog series include:

- Nmap
- Dnsrecon
- Netexec
- Other Linux command-line tools
- SMB null-sessions
- Anonymous LDAP binding
- Password Spraying
- LLMNR Poisoning
- SMB Relays
- IPv6 Takeovers
- Gaining Shell Access
- PowerShell AMSI Bypass
- Enumeration through RSAT, PowerView, Impacket, Bloodhound, Etc.
- Abusing DACL/ACE Permissions
- Constrained/Unconstrained Delegation
- Hash Cracking/Pass-the-Hash/Pass-the-Password
- AS-REP Roasting
- Kerberoasting
- Golden/Silver Ticket Attacks
- DCSync
- DPAPI/LSASS Dumping
- And Much More!

I will be using my own Active Directory homelab to carry out these tasks. If you’d like to follow along with my project, I’d highly recommend building your own lab. It’s a super beneficial learning experience that will give you hands-on AD administration and troubleshooting skills. I will *not* be showing you how to do that, *but* there are many Active Directory lab-building resources. All you need is a virtualization hypervisor such as VMWare Workstation or ProxMox, a guide if you’re a first time builder, and a solid chunk of time dedicated to setup.

Heath Adams, a.k.a, the Cyber Mentor, has an excellent [Hacking Active Directory For Beginners](https://www.youtube.com/watch?v=VXxH4n684HE) crash course on Youtube that I strongly recommend. It covers many of the same topics I will be discussing here, and it also includes a simple lab build. You can also find a myriad of other blog posts from cyber professionals detailing their own lab builds.

Additionally, if you don’t want to setup your own lab environment, you can find rooms on [Tryhackme](https://tryhackme.com/) or [Hackthebox](https://academy.hackthebox.com/course/preview/active-directory-enumeration--attacks) with virtualized networks and attack boxes already setup for you. Be mindful that some rooms are paid for while others you can find for free.

If you’re ready to get started, then you can check out the rest of my Active Directory posts by finding the `Active Directory` category under the `Categories` page.
