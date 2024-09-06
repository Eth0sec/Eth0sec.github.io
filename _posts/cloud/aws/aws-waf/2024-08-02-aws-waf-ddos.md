---
title: Keep Your Uptime High With AWS WAF DDoS Protections
excerpt: "DDoS attacks can cripple your site, but AWS WAF is your ally in maintaining uptime. Learn how to test and configure AWS WAF to fend off these attacks and keep your systems resilient."
categories:
  - cloud-networking
  - web-security
tags:
  - aws-waf
  - dos-attacks
header:
  teaser: /assets/images/teasers/aws-waf.jpg
featured: true
---

## The Battle for Uptime

A well-timed DDoS attempt can be ruinous. Imagine you just had an amazing product launch or your business relies on critical services being available at all times. Your web servers, humming along smoothly, suddenly get bombarded with a tsunami of traffic. It’s not the kind of traffic you want - this is Distributed Denial of Service (DDoS), designed to overwhelm your systems and knock them offline.

Extended downtime can cripple your service, alienate your users, and hit your bottom line hard. You need to be able to know how to stop it, and if you host in the cloud, AWS Shield might be a good choice for you. In this post, I’ll show you how to test your applications against DDOS attacks and harness the power of AWS WAF to keep your spirits high and your uptime even higher.

## Testing DDoS Using SlowHTTPTest

Usually denial of service type attacks attempt to overwhelm a server with unexpected high volume traffic, but there are other methods, and SlowHTTPTest can help us perform them.

SlowHTTPTest is an open-source tool designed to simulate various types of application layer denial-of-service (DoS) attacks, particularly those that exploit vulnerabilities in HTTP protocols. Unlike traditional DDoS attacks that overwhelm a server with a high volume of traffic, SlowHTTPTest focuses on generating slow, low-volume requests that tie up server resources, making it difficult for legitimate users to access the application.

### How SlowHTTPTest Works

SlowHTTPTest sends HTTP requests at a slow rate, keeping the server’s connections open for as long as possible without triggering timeouts. This method can exhaust the server’s resources, such as memory and connection slots, leading to service degradation or even a complete shutdown. The tool can be used to test several different types of slow attacks:

- **Slowloris Attack:** Maintains multiple open connections to the server by sending partial HTTP requests, keeping them alive and preventing the server from closing the connections.
- **Slow POST Attack:** Sends HTTP POST requests with large payloads at a slow rate. The server waits for the full payload before processing, causing it to hold the connection open.
- **Slow Read Attack:** Sends legitimate HTTP requests but reads responses very slowly, tying up server resources.

Let’s give it a shot. I’m going to target one of my EC2 DVWA web applications that I’ve used for each of these posts thus far with a Slowloris attack.

I’ll use something like this:

```bash
slowhttptest -d <your_instance_public_ip>:<your_port> -i 2 -r 148311 -c 9999
```

**Command breakdown:**

- `-d <ip_address>:8080`: Specifies the destination IP address and port number where the attack is directed. The server running at this address and port will be the target of the attack.
- `-i 2`: Sets the interval between follow-up HTTP headers or packets to 2 seconds. This means that after sending an initial request, SlowHTTPTest will wait 2 seconds before sending the next part of the request, keeping the connection open.
- `-r 148311`: Specifies the rate at which connections are created, which in this case is 148,311 connections per second. This is an extremely high rate, which can quickly exhaust the server’s resources.
- `-c 9999`: Indicates the number of connections to maintain, with 9,999 connections being opened and kept alive simultaneously.

Now that you have a better understanding of Slowloris attacks and how they work, execute the command on your attack machine. Wait for SlowHTTPTest to initialize, and you’ll see that it does not take long _at all_ for the application server to get knocked offline.

![SlowHTTPTest Slowloris Attack Demo](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/slowhttptest-ddos.jpg)

Go to your application and test the status of the website. You could try all you want, but it’s futile trying to access it. The server is unresponsive due to the amount of partial connections that are being kept open. You might even find that your target group instance undergoing the attack has been flagged as unhealthy after a short period of time has passed, further indicating the DDoS attack was successful.

It would be super easy for an attacker to take down critical infrastructure using the same method we just took, but it’s equally as easy to defend against using a managed rule group and a custom rate limiting rule.

## AWS IP Reputation List

In a previous post I explained the benefits of using Amazon’s IP Reputation list to stop requests from known malicious addresses, but it also has an additional DDoS protection rule as well.

Go to your WAF management console and access your web ACL. Click `Rules`, click `Add rules`, and then select `Add managed rule groups`. Under `AWS managed rule groups` find `Amazon IP reputation list` and click `Edit`.

You should find a rule titled `AWSManagedIPDDoSList`. This is a rule that filters traffic based on IP addresses known for DDoS attacks. By default it is set to count, but we want to change it to block. You can do this by selecting the `Override to Block` option. Now your managed rule set is taking care of DDoS protections…partially. For any IP addresses not included in that list, we can filter traffic by rate.

## Custom Rate-Limiting Rule

Rate limiting is a great way to catch DDoS since we can use it to regulate traffic and control the amount of requests made by any given user.

Go to the rules tab under your web ACL and choose `Add my own rules and groups`.

Follow these instructions to set up a rate-limiting rule:

1. Enter an appropriate name for the rule.
2. This time, make sure to select `Rate-based rule` rather than regular rule.
3. Pick whichever rate limit suits you. This will vary vastly depending on your own user needs. Choose something about 2 or 3 times higher than the amount of requests your average user would typically make. It would be wise to have an understanding of your access logs, amount of proxy traffic, and more stuff like that. For this example I will choose `1000`.
4. Choose `Source IP Address`. We will want to block by IP.
5. Choose `Consider all requests`.
6. If it didn’t need saying before, choose `block` for the action.
7. Save your rule and choose your priority.

![DDOS Rate Limit Rule Name](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos-rule.jpg)

![DDOS Rate Limit Rule Criteria](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos-criteria.jpg)

![DDOS Rate Limit Rule Action](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos-action.jpg)

![DDOS Rate Limit Rule Priority](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos-priority.jpg)

Now, this is all well and good to block normal DDoS attacks, but something like our Slowloris DDoS attack might slip through since it’s not strictly a rate-based attack. Slowloris attacks will send partial or malformed HTTP headers that are typically oversized compared to normal HTTP requests. These oversized headers are primarily what’s used to crash the site. We can add another simple rule that restricts packets with HTTP headers over a certain size.

## Custom Header Size Limit Rule

Follow these instructions to create the rule:

1. Name your rule something appropriate.
2. For the inspection type, select `All headers`. Leave the header options as is.
3. For match type, choose `Size greater than or equal to`.
4. Set the size in bytes to `4096`. Normally this byte size would not block valid headers. If your site uses Json Web Tokens (JWT) or custom x-headers that can be quite large, adjust as needed for your own application.
5. Set oversize handling to `Continue`.
6. Set the action to `Block`.
7. Save your rule and choose the priority.

![DDOS Header Size Limit Rule Name](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos2-rule.jpg)

![DDOS Header Size Limit Statement](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos2-statement.jpg)

![DDOS Header Size Limit Actions](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos2-action.jpg)

![DDOS Header Size Limit Priority](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/ddos2-priority.jpg)

Go back to your attack machine and try to run the Slowloris script once more. You should see that the connection failed with the exit status: `Cannot establish connection`. The rule successfully blocked the DDoS attempt.

![Failed Slowloris Attack Demo](/assets/images/posts/cloud/aws/aws-waf/aws-waf-ddos/slowhttptest-fail.jpg)

## Food For Thought

- **Know Your Traffic:** Analyze your traffic patterns to set realistic rate limits and header sizes that won’t block legitimate requests.
- **Monitor and Adjust:** Regularly review your AWS WAF rules and adjust them based on real-world traffic and attack patterns.
- **Combine Defenses:** Use AWS Shield alongside WAF for comprehensive DDoS protection.

That’s all she wrote. If you want to learn more about AWS WAF you can find my other posts on the topic under the `Categories` or `Tabs` page.
