---
title: Network Analysis - Ransomware
date: 2023-01-28
image: 
  path: https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/Wireshark_icon.svg/1200px-Wireshark_icon.svg.png
permalink: /writeups/network-analysis-ransomware
categories: [Write-Ups, BTLO, Challenges]
tags: [BTLO, DFIR]
---

## Scenario
ABC Industries worked day and night for a month to prepare a tender document for a prestigious project that would secure the company’s financial future. The company was hit by ransomware, believed to be conducted by a competitor, and the final version of the tender document was encrypted. Right now they are in need of an expert who can decrypt this critical document. All we have is the network traffic, the ransom note, and the encrypted ender document. Do your thing Defender!​

`Wireshark`{:.info} `Tshark`{:.info} `TCPDump`{:.info}

## Challenge Submission

### What is the operating system of the host from which the network traffic was captured? (Look at Capture File Properties, copy the details exactly)

1. Open file with *Wireshark*
2. *Statistics*
3. *Capture File Properties*

OS: **32-bit Windows 7 Service Pack 1, build 7601**

### What is the full URL from which the ransomware executable was downloaded?

1. Always with *Wireshark*
2. *Statistics*
3. *HTTP*
4. *Requests*
--> **http[:]10.2.2.15:8000/safecrypt.exe**

### Name the ransomware executable file?

Obviously **safecrypt.exe**. We found it at the previous question.

### What is the MD5 hash of the ransomware?

I searched malware's name on Google and I found it:
**4a1d88603b1007825a9c6b36d1e5de44**

### What is the name of the ransomware?

Upload md5 hash on [VirusTotal](https://www.virustotal.com/){:target="_blank"}:  **TeslaCrypt**

### What is the encryption algorithm used by the ransomware, according to the ransom note?

Look into png file and you will find the answer: **RSA-4096**.

### What is the domain beginning with ‘d’ that is related to ransomware traffic?

Find the answer looking into VirusTotal Community section: **dunyamuzelerimuzesi.com**

### Decrypt the Tender document and submit the flag

Download and use the [Trend Micro Ransomware File Decryptor](https://success.trendmicro.com/dcx/s/solution/1114221-downloading-and-using-the-trend-micro-ransomware-file-decryptor?language=en_US). 
