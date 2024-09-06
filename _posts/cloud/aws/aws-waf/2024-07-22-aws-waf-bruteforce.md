---
title: How to Brute Force Login Pages and Fortify Them with AWS WAF
excerpt: "Master the art of defending your login pages against brute-force attacks using AWS WAF. Learn how to simulate these attacks and set up powerful rules to protect your application from unauthorized access."
categories:
  - cloud-networking
  - penetration-testing
  - web-security
tags:
  - aws-waf
  - brute-force-attacks
header:
  teaser: /assets/images/teasers/aws-waf.jpg
---

If you haven’t done so already, I would highly recommend checking out my [AWS WAF post on preventing XSS attacks](http://localhost:4000/cloud-networking/web-security/penetration-testing/aws-waf-xss/) first. I cover how to setup your own Web ACL if you need guidance on how to do so.

## Cracking the Code

Let’s be honest, threat actors are always going to take the path of least resistance to break into your web application and brute-forcing is so simple and easy to do. Basic HTTP authentication brute forcing takes a matter of seconds to perform when you have handy word lists filled with millions of passwords. If you lack strong account lockout policies and rate limiting then your web applications will get owned by attackers.

Luckily, AWS WAF makes it easy to implement smart defenses that stop brute-force attacks in their tracks. In this post, I will discuss how you can simulate brute-force attacks for testing and then use web ACL rules to prevent attackers from ever getting the chance to guess your passwords.

## Testing Brute-Force Attacks

Brute force attacks are those that attempt to systematically guess user and password combinations to break into systems. Most modern brute force attacks are automated with software and use dictionaries filled with thousands to millions of common passwords. Grab a Kali Linux attack machine or something comparable and let’s use a program called Hydra to simulate a brute-force attack against DVWA. It’ll be no use to try cutting off these hackers' heads.

![Get Up On the Hydra's Back](/assets/images/posts/cloud/aws/aws-waf/aws-waf-bruteforce/hailhydra.webp)

Hydra is a powerful and popular network login cracker that supports numerous protocols and is widely used for penetration testing and security assessments. It comes pre-installed with Kali Linux by default.

Boot into Kali and navigate to the directory `usr/share/wordlists` to find a bunch of awesome pre-installed wordlists for you to use for brute-force testing. I’ll be picking `rockyou.txt`. It’s one of the most famous password lists available. Here is the Hydra command you need to execute:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt <your-load-balancer-1234567890>.<your_region>.elb.amazonaws.com http-post-form "/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed"
```

Let me break this down for you:

**hydra:**

- The command to run the Hydra tool.

**-l admin:**

- **-l:** Specifies a single username to use for the attack.
- **admin:** The username Hydra will try with each password in the wordlist. `admin` is the default DVWA user credential.

**-P /usr/share/wordlists/rockyou.txt:**

- **-P:** Specifies the path to the password wordlist.
- **/usr/share/wordlists/rockyou.txt:** The file containing a list of passwords that Hydra will try for the specified username.

**<your-load-balancer-1234567890>.<your_region>.elb.amazonaws.com:**

- The public DNS name of your AWS load balancer. Replace this placeholder with the actual DNS name of your load balancer. Hydra will send the login requests to this address.

**http-post-form:**

- Specifies the type of request Hydra will use. In this case, it indicates that Hydra should perform a brute force attack on a web form that uses HTTP POST.

**"/login.php:username=^USER^&password=^PASS^&Login=Login:Login failed":**

- This is the format of the HTTP POST request Hydra will use:
  - **/login.php:** The path to the login page where the form is located.
  - **username=^USER^:** The form parameter for the username field, where ^USER^ is replaced by the username specified with -l.
  - **password=^PASS^:** The form parameter for the password field, where ^PASS^ is replaced by each password in the wordlist.
  - **&Login=Login:** The value of the submit button in the form (this may vary based on the form’s HTML).
  - **Login failed:** The string Hydra looks for in the response to determine if the login attempt failed. If this string is found, the login is considered unsuccessful.

Now that it’s understood what this command does, let’s execute it and see what happens.

![Hydra Successful Brute Force](/assets/images/posts/cloud/aws/aws-waf/aws-waf-bruteforce/hydra-scan.jpg)

Hydra was able to find not just one, but *sixteen*, valid credentials for the DVWA admin account. With these credentials, attackers can gain unauthorized access to your application, steal data and intercept communications, escalate privileges, compromise your systems through malware installations, and create persistent access to your application. One small brute-force attack leads to huge outcomes as demonstrated above. Let’s now see how a WAF can stop this.

## Creating a Brute Force Rule using Managed Rule Sets

It’s very easy to block brute force attacks with managed rule sets. What are managed rule sets you might ask? Managed rule sets are pre-configured sets of rules provided by Amazon and other vendors. These rule sets are designed to simplify the process of protecting your web applications from common threats by offering pre-defined security rules that can be easily applied to your web application firewall. They offer support for common web security vulnerabilities and OWASP Top Ten threats.

Go to your WAF management console to access your web ACL, or create one if you haven’t done so already. Go to the **Rules** tab and select **Add managed rule groups** from the dropdown menu to create a managed rule set. AWS offers an **Account takeover prevention** rule set that has brute force protections built-in. Simply click the **Add to web ACL** toggle and then click **Edit** to modify the rules.

**QUICK NOTE:** The cost to use this rule set is $10.00 USD a month prorated hourly. That means you’ll be charged the monthly rate (10) divided by the total hours in a month (720) per hour. This amounts to $0.0139 per hour of usage. Make sure to nuke the rule set after you test it in this lab to avoid incurring more fees than necessary.

Follow these instructions to configure the rule group:

1. Leave the version at the default setting
2. Leave the scope of inspection at default settings.
3. Add the URL for the login path, for DVWA it should be `/login.php`.
4. Change the payload type to **FORM_ENCODED**.
5. The username and password field should be set to **username** and **password** respectively.
6. Leave the account takeover prevention rules section as is. The rules will automatically be set to block. Many of these rules help prevent brute force attacks, specifically the **VolumetricIpHIgh** which will block certain IP addresses who make too many failed login requests.
7. Click on **Save rule**.
8. Click **Add rules**.
9. Review the rule priority and then click **Save**.

![Account Takeover Prevention Config](/assets/images/posts/cloud/aws/aws-waf/aws-waf-bruteforce/atp-config.jpg)
![Account Takeover Prevention Rules](/assets/images/posts/cloud/aws/aws-waf/aws-waf-bruteforce/atp-rules.jpg)
![Account Takeover Prevention Config](/assets/images/posts/cloud/aws/aws-waf/aws-waf-bruteforce/atp-config.jpg)

It’s as simple as that. Go to DVWA and change the admin password to something much stronger like `Sup3rStr0ngP@ss321#`, then head back to Kali and attempt the same brute force attack as before. You’ll notice that the attempts eventually hang and fail because we are making too many requests to the server from the same IP address. The rule is successfully blocking our brute force attempts!

![Failed Hydra Brute Force Attempt](/assets/images/posts/cloud/aws/aws-waf/aws-waf-bruteforce/hydra-scan2.jpg)

## Food For Thought

In this post, you’ve learned how to combine a strong password policy with managed WAF rule groups to confidently block brute force attempts. You can take these additional steps to secure your application even more against brute force attacks:

- **Consider Custom Rule Sets:** Managed rule sets are a great start, but don’t forget that custom rules tailored to your specific application can offer an extra layer of protection. Think about adding rate-limiting rules or IP blacklists for known threats.
- **Regularly Update Your Password Policies:** Even with a strong password, it’s wise to periodically review and update your password policies. Consider implementing multi-factor authentication (MFA) and reCAPTCHA for additional layers of security.
- **Monitor and Analyze Logs:** Keep an eye on your AWS WAF logs and analyze them for unusual patterns or failed login attempts. This proactive approach can help you identify and mitigate threats before they escalate.

That concludes this post. You can find the rest of my security and networking posts under the **categories** and **tags** pages.
