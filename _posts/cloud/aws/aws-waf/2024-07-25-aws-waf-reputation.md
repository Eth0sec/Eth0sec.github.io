---
title: Block Suspicious IPs with AWS WAF’s IP Reputation & Anonymous IP Rules
excerpt: "Tired of manually blocking suspicious IPs? Discover how to automate the process using AWS WAF’s IP Reputation and Anonymous IP Lists to keep your web app safe and sound"
categories:
  - cloud-networking
  - web-security
tags:
  - aws-waf
header:
  teaser: /assets/images/teasers/aws-waf.jpg
---

## Quit Acting Suspicious

Ever spotted some suspicious activity in your traffic logs, with certain IP addresses making questionable requests to your web app? You head over to VirusTotal or ThreatFeeds, only to find that these IPs are flagged as malicious by multiple security vendors. Sure, you could block that one IP, but there’s a more efficient way. By setting up a web ACL rule, you can automatically block all suspicious traffic based on their inclusion in threat intelligence feeds. It’s simple to set up, and I want to show you how.

## Amazon IP Reputation List and Anonymous IP List

There are a few managed rule groups that we can use for this task. Two of them being the Amazon IP reputation list and the Anonymous IP list. What’s the difference between the two?

**Amazon IP Reputation List:**

This rule group is about keeping out bad actors. It automatically blocks traffic from IP addresses that are known to be associated with bots, scrapers, and attackers. AWS keeps this list updated regularly, so you’re protected from emerging threats without lifting a finger.

**Anonymous IP List:**

This rule group focuses on blocking traffic coming from anonymized IP addresses—think VPNs, Tor nodes, and proxies. While not all anonymous IPs are malicious, they do obscure the true source of the traffic, which can be a red flag in certain scenarios. If you’re looking to tighten security by preventing users from hiding their tracks, this rule group has you covered.

Using a combination of the two is your best bet for filtering out malicious activity. Let’s set up our rules now.

## Creating the Rules

Go to your Web ACL and add two new managed rules, toggle on both `Amazon IP reputation list` and `Anonymous IP list`. Click `Add rules`, sort them in the rule priority list, and then click `Save`. That’s…really all there is to it. You can always choose which rules in the rule sets get used through overrides, but why turn off extra security features?

![Amazon Reputation And Anonymous Lists](/assets/images/posts/cloud/aws/aws-waf/aws-waf-reputation/repanon-rules.jpg)

![Amazon Reputation And Anonymous List Priority](/assets/images/posts/cloud/aws/aws-waf/aws-waf-reputation/repanon-priority.jpg)

## Testing Anonymous IP Rules

There’s no good way to test the Amazon reputation list rule group, but you can test the anonymous IP one. All you will need is a VPN. Choose a different location, head to your load-balanced application, and try to access it. You should find that you’re being blocked or timed out and the rule is working as it should.

![DVWA Connection Timeout](/assets/images/posts/cloud/aws/aws-waf/aws-waf-reputation/dvwa-timeout.jpg)

## Food For Thought

- **Layer Your Defenses:** Adding these WAF rules is just the first step. Explore other managed rule groups or custom rules to create a multi-layered defense. What additional threats could you mitigate?
- **Monitor and Adapt:** Your threat landscape can change quickly. Regularly monitor the effectiveness of these rules and adjust as needed. Are there new patterns of suspicious behavior emerging?
- **Evaluate Impact:** After implementing the rules, assess how they affect your application’s performance and user experience. Are you blocking the right traffic without hindering legitimate users?

Another shorter post for today, so maybe check out a few other of my posts while you’re at it? You can find a variety of categorized and tagged posts under the `categories` and `tags` pages.