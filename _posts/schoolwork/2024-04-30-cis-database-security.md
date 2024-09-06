---
title: Selected Database Security CIS Coursework - UofL
excerpt: "My portfolio of work from UofL course CIS 483 - Database Security"
categories:
  - active-directory
  - database-security
  - penetration-testing
  - web-security
tags:
  - burpsuite
  - group-policy
  - nmap
  - powershell
  - sql
  - sqli-attacks
header:
  teaser: /assets/images/teasers/uofl.jpg
---

# Selected Database Security CIS Coursework - UofL

All of these assignments were completed as part of the `CIS 483 - Database Security` course for my Computer Information Systems Cybersecurity degree at the University of Louisville.

## Completed Coursework

### Database Security and System Vulnerabilities Analysis

This assignment delves into various tasks related to database attacks and defense mechanisms. It involves exploiting vulnerabilities such as SQL injection to manipulate database queries, creating unauthorized accounts, and executing unauthorized commands. Additionally, it explores the creation of malicious tables in databases and accessing system files through directory traversal. Furthermore, the assignment addresses monitoring and detecting suspicious activities by capturing screenshots of system processes.

<iframe src="/assets/docs/Schoolwork/Homework3_102.pdf" width="100%" height="600px"></iframe>

### Active Directory Management and Group Policy Analysis

This assignment focuses on Active Directory (AD) management and Group Policy Objects (GPOs), emphasizing tasks related to user and group administration, security policies enforcement, and PowerShell scripting. It involves creating domain users and groups within specific Organizational Units (OUs), configuring user memberships, and validating user logins. Additionally, it addresses the implementation of password policies through GPOs and their impact on SQL Server authentication. Furthermore, it explores the concepts of account creation automation using PowerShell cmdlets.

<iframe src="/assets/docs/Schoolwork/Homework4_102.pdf" width="100%" height="600px"></iframe>

### Database Management & Permissions Assignment Analysis

This assignment involves various tasks related to database management and permissions. It covers topics such as assigning user mappings to fixed database roles, creating custom database roles, creating application roles, and assigning permissions. Each task includes steps to verify results and explanations for successes or failures based on permission settings.

<iframe src="/assets/docs/Schoolwork/Homework5_102.pdf" width="100%" height="600px"></iframe>

### SQL Injection Vulnerability Assessment Using BurpSuite

This lab focuses on exploring SQL injection vulnerabilities using BurpSuite. It involves retrieving hidden data through HTTP interception and blind SQL injection.

<iframe src="/assets/docs/Schoolwork/Lab3_Epperson.pdf" width="100%" height="600px"></iframe>

### Windows SID and PowerShell Scripting

This lab focuses on understanding Security Identifiers (SIDs) in Windows environments as well as Powershell scripting techniques. Covered tasks include obtaining the SID of the current login using WMIC commands and from the Registry, obtaining SIDs through SQL Server, comparing admin login SIDs of SQL Server and Windows Server, analyzing SID differences for specific accounts, and creating a basic PowerShell script that outputs name and date.

<iframe src="/assets/docs/Schoolwork/Lab4_Epperson.pdf" width="100%" height="600px"></iframe>

### Understanding Ownership Chains in SQL Server

This lab delves into understanding ownership chains in SQL Server. It demonstrates how ownership chaining enables indirect access to objects within the same schema but requires explicit permissions for direct access to underlying objects.

<iframe src="/assets/docs/Schoolwork/Lab5_Epperson.pdf" width="100%" height="600px"></iframe>

### MySQL Security Assessment and Penetration Testing

This collaborative, group project involved conducting penetration testing on a MySQL server to assess its security vulnerabilities. Tasks included performing a comprehensive Nmap scan, brute-forcing logins, obtaining the MySQL version, enumerating MySQL users, dumping password hashes, and dumping the database schema. The team successfully identified open ports, discovered MySQL version information, extracted user accounts and their privileges, retrieved password hashes, and identified the database schema with 46 tables containing various types of information.

<iframe src="/assets/docs/Schoolwork/CIS483PenTest-Team10.pdf" width="100%" height="600px"></iframe>
