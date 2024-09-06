---
title: Stop SQL Injections With WAF Managed and Custom WAF Rules
excerpt: "I’ll show you how to protect your web apps from SQL injections with AWS WAF. Learn how to use managed and custom rules to keep your apps secure."
categories:
  - cloud-networking
  - web-security
tags:
  - aws-waf
  - sqli-attacks
header:
  teaser: /assets/images/teasers/aws-waf.jpg
---

## SQL? I Preferred the Original

SQL injections remain at the top of the OWASP Top Ten in 2024. They’re one of the most common web vulnerabilities and remain formidable. If you’ve ever had a rogue query slip through and wreak havoc on your back-end database, you know the damage these attacks can do. If you host in the cloud, your web applications could use some protection from AWS WAF, and luckily, there’s a lot of managed rule groups to choose from for stopping injections. Why stop there though, you can also make your own rules to layer on protections. I’ll show you how you can do both.

## Testing SQL Injections

Before you do anything else, you should probably check to see if your application is vulnerable to SQLi. I’m running Damn Vulnerable Web Application through a couple of load-balanced EC2 instances. This should be easy. We already know the app is vulnerable.

If you’ve been following along with these posts and have your own DVWA instances the follow along. Navigate to DVWA and choose `SQL Injection`. You’ll see a User ID form with a submission button. Let’s take a look at the [PHP source code](https://github.com/digininja/DVWA/tree/master/vulnerabilities/sqli/source).

There’s a lot to digest there. I’ll break it down:

### SQL Injection Vulnerability in Source Code

**Problematic Code:**

```php
$query = "SELECT first_name, last_name FROM users WHERE user_id = '$id';";
```

Here’s what it does:

- `%\' and 1=0`: Ends the current string context and adds a condition that is always false to bypass the original query’s logic.
- `union select null, concat(...)`: Combines the original query results with data from the users table. `concat(...)` formats the user information with newlines.
- `from users`: Specifies the users table to extract data from.
- `#`: Comments out the rest of the query to prevent errors.

![Successful SQLi Injection on DVWA](/assets/images/posts/cloud/aws/aws-waf/aws-waf-sqli/dvwa-sqli.jpg)

With this we are able to steal the MD5 hashes for each user in the table. We can crack these hashes to gain passwords, compromise accounts, steal data, and ruin user trust and reputation. It’s not a good look. Let’s see what we can do about with our WAF.

## SQLi Protection With WAF ACLs

Go to your web ACL and add a new managed rule group under `'AWS managed rule groups'` called `'SQL database'`. These are pre-configured rules that Amazon has compiled to prevent SQL injection attacks. It’s as simple as plug and play. All you need to do is toggle it to activate it. You can click `Edit` to see what the rules are, how they work, and override them if you wish. Every rule is set to block certain behaviors by default like query string inputs in URI paths. Click `'Add rule'` and organize the priority list to your specification.

![SQL Rule Group](/assets/images/posts/cloud/aws/aws-waf/aws-waf-sqli/sql-rules.jpg)

![SQL Rule Group Priority List](/assets/images/posts/cloud/aws/aws-waf/aws-waf-sqli/sql-priority.jpg)

Go back to DVWA and try to execute the same SQLi attack as before. You’ll now see a 403 forbidden error! The AWS database protection rule group is the real deal, but you can also set up your own custom rule to target injections. Let’s try this method as well.

## Custom SQLi Protection Rule

Disable the managed SQL rule group for the time being. In your ACL, click `'Add rules'` and then `'Add my own rules and rule groups'`.

Follow these instructions to setup your rule:

1. Name your rule something appropriate for the task.
2. Leave `'If a request'` set to `'matches the statement'`.
3. Set the inspection type to `'All query parameters'`.
4. Set the match type to `'Contains SQL injection attacks'`.
5. Choose the `'URL decode'`, `'HTML entity decode'`, `'Lowercase'`, `'Compress whitespace'`, `'Replace comments'`, and `'Normalize path'` in that order.
6. Set the sensitivity level to `'High'`.
7. Leave the action on `'block'`.
8. Save your rule and set the rule priority.

![Custom SQLi Rule Name](/assets/images/posts/cloud/aws/aws-waf/aws-waf-sqli/customsql-rule.jpg)

![Custom SQLi Rule Statement](/assets/images/posts/cloud/aws/aws-waf/aws-waf-sqli/customsql-statement.jpg)

![Custom SQLi Rule Action](/assets/images/posts/cloud/aws/aws-waf/aws-waf-sqli/customsql-action.jpg)

If you go back to your application and try to inject a SQL query into the input form you should see a 403 error, same as before with the managed rule groups. You can either pick between keeping the managed rules or your custom rule, or better yet double them. Just remember that rule order matters. Those at the top of the list will take greater priority. Using a mix of managed and custom rules can help you make more granular choices where needed while keeping management simple in other areas.

## Food For Thought

- **Fine-Tune SQLi Rules:** Customize your AWS WAF SQL injection rules to target specific query patterns that are unique to your application.
- **Leverage Rule Testing:** Use the AWS WAF logging and testing tools to simulate SQLi attacks and ensure your rules are effectively blocking malicious queries.
- **Stay Updated on Bypasses:** Keep an eye on emerging WAF bypass techniques for SQL injections and adjust your rules to stay ahead of attackers.

I hope you were able to learn something valuable about SQLi and WAF from this post! If you liked this one, maybe check out my other posts? You can find a variety of networking and security content under the `'categories'` or `'tags'` pages.
