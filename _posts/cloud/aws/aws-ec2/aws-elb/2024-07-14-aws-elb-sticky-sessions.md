---
title: Enhancing Session Persistence with ALB Sticky Sessions
excerpt: "Demonstration of an application load balancer session flaw and remediation steps using the sticky sessions feature for load-balanced target groups."
categories:
  - cloud-networking
tags:
  - aws-ec2
  - aws-elb
header:
  teaser: /assets/images/teasers/aws-elb.jpg
---

## Painfully Nonpersistent

Here’s the setup:

You have two separate web applications, `App 1` and `App 2` running on two different EC2 instances, `Instance1` and `Instance2`. Both applications are served through a single AWS Application Load Balancer (ALB). Seems fine, right? However, this simple setup can be problematic and result in some headaches for end-users. Let’s take a look at why that is.

## What Can Happen?

### 1. Apparent Successful Login

**Situation**: The user logs into `App 1` successfully on `Instance 1`, but when they use the ALB to access the application, they’re redirected to `App2` on `Instance 2`.

**Possible Outcome**:

- If `App 2` on `Instance 2` does not have access to the session information from `App 1`, the user might be redirected to `App2` and see it as if they have logged in, but `App 2` will not recognize their session from `App1`.
- `App 2` might not have any indication that the user is already logged in on `App 1`. Thus, it might prompt the user to log in again or show an interface indicating that they are not logged in.

### 2. Inability to Login

**Situation**: The user logs into `App 1` on `Instance 1`, and then accesses the ALB, which directs them to `App 2` on `Instance 2`.

**Possible Outcome**:

- When the user is routed to `App 2`, `App 2` has no session information from `App 1`. This means that the user is effectively logged out when they are directed to `App 2`.
- The user might be unable to access the features or content available to logged-in users because `App 2` does not recognize their session or credentials, leading to a login failure or prompt.

That my friend, has to do with a lack of session persistence, or sticky sessions as they are called in AWS. Well…what are they? Sticky sessions are used by application load balancers to ensure that a user’s session is consistently routed to the same server or instance throughout their interaction with a web application.

It ensures problems like the examples above can’t happen to an end-user, and is a crucial concept for applications that require user data to be maintained across multiple requests. User sessions on e-commerce sites and login-based applications rely on sticky sessions.

## How Do Sticky Sessions Work?

Sticky sessions with AWS’s Application Load Balancer (ALB) are all about keeping things consistent for your users. Here’s the lowdown on the two stickiness methods you can choose.

### 1. App-Based Cookies

Your app can issue its own AWSALB cookie to track which instance should handle a user’s requests. This cookie lets the load balancer know exactly where to send the traffic, making sure users stay with the same server throughout their session.

### 2. Load Balancer-Generated Cookies

ALB: If you let the load balancer handle it, it automatically sets an AWSALB (or AWSALBCORS for CORS setups) cookie. This cookie keeps track of the instance serving the user, so they’re routed back to the same one without any extra fuss.

Here’s a simplified look at how a load balancer-generated cookies work.

![Sticky Session Infographic](/assets/images/posts/cloud/aws/aws-ec2/aws-elb/aws-elb-sticky-sessions/stickiness.drawio.png)

## Setting Up Sticky Sessions

Follow the steps below to turn on stickiness for your target group of instances:

1. Under `Load Balancing`, choose `Target Groups`, then `Attributes`, and finally click `Edit`.
2. Scroll down to the `Target selection configuration` and check the box to turn stickiness.
3. Leave the type as `Load balancer generated cookie`.
4. Choose whatever stickiness duration you desire.
5. Leave `Cross-zone load balancing` as is.
6. Click `Save changes`.

![Target Group Attributes Window](/assets/images/posts/cloud/aws/aws-ec2/aws-elb/aws-elb-sticky-sessions/tg-attributes.jpg)

![Target Selection Configuration](/assets/images/posts/cloud/aws/aws-ec2/aws-elb/aws-elb-sticky-sessions/target-selection.jpg)

It’s as simple as that. Session stickiness should now be active for all instances of the target group.

## Conclusion + Food For Thought

Navigating multiple web applications behind an ALB can be a balancing act of its own. To keep user sessions smooth and seamless, here’s what you should focus on:

- **Sticky Sessions**: Think of them as a GPS for user traffic — once a user is connected to a specific instance, they stay on the same path for the rest of their session.
- **Session Harmony**: You want your sessions to be in harmony. Applying consistent strategies across your apps helps them avoid dissonance.
- **ALB Configuration**: Double-check your ALB setup to make sure it’s directing your traffic smoothly like butter 🧈.

That’s all for this one! You can find a variety of my security and networking posts under the `categories` and `tags` pages.