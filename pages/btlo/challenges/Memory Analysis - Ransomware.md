#BTLO #Challenges #DFIR 

---
layout: article
title: Malware Analisys - Ransomware
aside:
  toc: true
sidebar:
  nav: btlo
cover: /img/***
tags:
- BTLO
- Challenges
---
## Scenario
The Account Executive called the SOC earlier and sounds very frustrated and angry. He stated he can’t access any files on his computer and keeps receiving a pop-up stating that his files have been encrypted. You disconnected the computer from the network and extracted the memory dump of his machine and started analyzing it with Volatility. Continue your investigation to uncover how the ransomware works and how to stop it!

## Write-up
To complete this challenge I used [volatility](https://github.com/volatilityfoundation/volatility3) tool version 3, the the world's most widely used framework for extracting digital artifacts from volatile memory (RAM) samples.

### Q1. Run “vol.py -f infected.vmem --profile=Win7SP1x86 psscan” that will list all processes. What is the name of the suspicious process?

In Volatility 3 `--profile` option was deprecated so you can execute it directly this: 
`python3 ./vol.py /f infected.vmem windows.psscan`

![[Pasted image 20230110172059.png]]

### Q2. What is the parent process ID for the suspicious process?

You can answer to this question launching tha same volatylity plugin (`psscan`) and look for **PPID** column referred to the suspicious process: **2732**

### Q3. What is the initial malicious executable that created this process?

It has the same **PID** of the previous **PPID**: `or4qtckT.exe`

### Q4. If you drill down on the suspicious PID (vol.py -f infected.vmem --profile=Win7SP1x86 psscan | grep (PIDhere)), find the process used to delete files

For Volatility 3, I launched `python3 ./vol.py -f infected.vmem windows.psscan | grep 2732` and below there is the output.

![[Pasted image 20230110173257.png]]

`taskdl.exe` was the process used to delete files.

### Q5. Find the path where the malicious file was first executed

You can just launch `python3 ./vol.py -f infected.mem windows.dlllist | grep or4qtckT.exe` and you will find the path where it was first executed.

![[Pasted image 20230110174032.png]]


### Q6. Can you identify what ransomware it is? (Do your research!)

By now it's simple understand what kind of ransomware is it: **WannaCry**.

### Q7. What is the filename for the file with the ransomware public key that was used to encrypt the private key? (.eky extension)

To asnswer this question I launched `python3 ./vol.py -f infected.mem windows.filescan | grep .eky` and I found the filename of the key.

![[Pasted image 20230110174439.png]]

