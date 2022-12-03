---
layout: article
title: Shiba Insider
aside:
  toc: true
cover: https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/shiba/dog.jpeg
---
Below a small walkthrough about **Shiba Insider** challenge on [BTLO](https://blueteamlabs.online/).

## Questions
![Alt text](https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/shiba/0.png)

### What is the response message obtained from PCAP file?
To answer this question I simply opened the pcap file with Wireshark and I followed the only HTTP stream available.

![Alt text](https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/shiba/1.png)

### What is the password of ZIP file?
Message said "use your own password", so I have seen "Authorization" header on http request and It was a base64 string. I decoded it with CyberChef and I found the password, because it's the same of the authentication.

![Alt text](https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/shiba/2.png)

![Alt text](https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/shiba/2.1.png)

I unzipped `file.zip` and in `README` file I read that this is the only password, so to the next question I aswered **no**.

### The name of a widely-used forensics tool
I imagined was `exiftool` and it was right. Too simple!

###  Name and value of interest information in image's metadata
I opened the image with `exiftool`.
```
exiftool ssdog1.jpeg
```
I discovered that information has been hidden with `steghide`.
`Technique:Steganography`

![Alt text](https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/shiba/3.png)

### ID 
To retrieve hidden information I used `steghide`.
```
steghide extract -sf ssdog1.jpeg
```
I omitted a passphrase and I obtained `idInsider.txt`, I found the ID!

![Alt text](https://raw.githubusercontent.com/z3f1r0/z3f1r0.github.io/master/img/shiba/4.png)

### What is the profile name of the attacker?
I spent about an hour to answer this question and I don't want to give you the answer because I think it is more challenging discover it to yourself.
I'll give you one little hint. Try to watch at your own profile. ;-)

---

I like very much [Blue Team Labs Online](https://blueteamlabs.online/). I think it is a funny platform where everyone can exercise  their cybersecurity skills. I started to make "easy" challenges but I'll go on and the game will be harder!
