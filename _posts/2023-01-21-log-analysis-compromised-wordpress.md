---
title: Log Analysis - Compromised Wordpress
date: 2023-01-21
image:
  path: https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcT8XBsRa-TGb1pIQyD5nGmbgIzTgxS6OG_Vnw&usqp=CAU
permalink: /writeups/log-analysis-compromised-wordpress
categories: [Write-Ups, BTLO, Challenges]
tags: [BTLO, DFIR]
---

## Scenario
One of our WordPress sites has been compromised but we're currently unsure how. The primary hypothesis is that an installed plugin was vulnerable to a remote code execution vulnerability which gave an attacker access to the underlying operating system of the server.

## Challenge Submission

### Q1. Identify the URI of the admin login panel that the attacker gained access to (include the token)
To find the answer you just have launch `cat access.log | grep .php?` and you will find the correct string: `wp-login.php?itsec-hb-token=adminlogin`

### Q2.  Can you find two tools the attacker used?
I was looking in `access.log` grepped with `wp-login.php` and fortunately I read the string "`sqlmap/1.4.11#stable (http://sqlmap.org)`", then I tried to grep for another famous tool for Wordpress exploiting: `wpscan`. And I was right. Tools were `sqlmap` and `wpscan`.

### Q3. The attacker tried to exploit a vulnerability in ‘Contact Form 7’. What CVE was the plugin vulnerable to? (Do some research!)
Simply you can write on Google "Contact Form 7 cve" and you will find your answer.

### Q4. What plugin was exploited to get access?
With the following command `cat access.log | grep /plugins/ | cut -d "/" -f6 | sort | uniq` you can enumerate all the plugins in the log file. Making an internet research you can find that the only vulnerable plugin is **simple-file-list** version 4.2.2.

### Q5. What is the name of the PHP web shell file?
You can find the name of the webshell launching `cat access.log | grep .php` and looking for strange php files. During my analysis I found a `fr34k.php` file and it was the webshell's name.

### Q6. What was the HTTP response code provided when the web shell was accessed for the final time?
You can launch `cat access.log | greap fr34k.php` and look to the final http response code: **404**
