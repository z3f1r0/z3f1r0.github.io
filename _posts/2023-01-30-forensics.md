---
title: Forensics Write-Up
date: 2023-01-30
image: 
  path: https://tryhackme.com/img/favicon.png
categories: [Write-Ups, TryHackMe]
tags: [TryHackMe, DFIR, volatility]
---

Room link --> [https://tryhackme.com/room/forensics](https://tryhackme.com/room/forensics){:target="_blank"}.
---

## Task 1 - Volatility forensics

*This is a memory dump of the infected system. Download the file attached to this Task.
The MD5 hash of the uncompressed file is: **ba44c4b977d28132faeb5fb8b06debce***

For this room I used [volatility 3](https://github.com/volatilityfoundation/volatility3){:target="_blank"}.

*Reference* --> [**Volatility 3 useful commands list**](https://dfir.science/2022/02/Introduction-to-Memory-Forensics-with-Volatility-3){:target="_blank"}.

### Verify md5 hash

`md5sum victim.raw` 
> ba44c4b977d28132faeb5fb8b06debce  victim.raw

### What is the Operating System of this Dump file? (OS name)

In Volatility 3 `--profile` option was deprecated so you can execute `vol.py -f victim.raw windows.info`.
> **windows**

### What is PID of SearchIndexer?

`vol.py -f victim.raw windows.pslist | grep SearchIndexer` (remove grep to see effective output layout).
> **2180**

### What is the last directory accessed by the user? (The last folder name as it is?)

Use previous version of volatility and launch `volatility -f victim.raw --profile=Win7SP1x64 shellbags | sort -k 6` like in this [write-up](https://github.com/emirfattoum/Forensics/blob/main/Memory%20Forensics/TryHackMe_Forensics_Walkthrought.pdf){:target="_blank"}.
> **deleted_files**

## Task 2

*Dig a little more...*

### There are many suspicious open ports; which one is it? (ANSWER format: protocol:port)

`vol -f victim.raw windows.netscan` 
> **udp:5005**

### Vads tag and execute protection are strong indicators of malicious processes; can you find which they are? (ANSWER format: Pid1;Pid2;Pid3)

We can use *malfind* plugin which lists process memory ranges that potentially contain injected code.

`vol -f victim.raw windows.malfind`
> **1860;1820;2464**

## Task 3 - IOC SAGA

*In the previous task, you identified malicious processes, so let's dig into them and find some Indicator of Compromise (IOC). You just need to find them and fill in the blanks (You may search for them on VirusTotal to discover more details).*

First-of-all dump malicious processes.

I found answers into **1820** PID: `vol -f victim.ram windows.memmap --pid 1820 --dump`.

`strings pid.1820.dmp | grep www.go | grep .ru`
> **www.goporn.ru**

`strings pid.1820.dmp | grep www.i | grep .com`
> **www.ikaka.com**

`strings pid.1820.dmp | grep www.ic | grep .com`
> **www.icsalabs.com**

`strings pid.1820.dmp | grep 202. | grep .233.`
> **202.107.233.211**

`strings pid.1820.dmp | grep .200. | grep .164`
> **209.200.12.164**

`strings pid.1820.dmp | grep 209. | grep .190.`
> **209.190.122.186**

`vol -f victim.raw windows.envars --pid 2464`
> **OANOCACHE**
