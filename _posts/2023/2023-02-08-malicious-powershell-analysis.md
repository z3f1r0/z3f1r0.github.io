---
title: Malicious Powershell Analysis
date: 2023-02-08
cover: https://miro.medium.com/max/786/0*Q-sW75vMp0rrOuv_.webp
categories: [Write-Ups, BTLO, Challenges]
tags: [BTLO, DFIR, powershell]
---

For this challenge I used [CyberChef](https://gchq.github.io/CyberChef/).

## Scenario
Recently the networks of a large company named GothamLegend were compromised after an employee opened a phishing email containing malware. The damage caused was critical and resulted in business-wide disruption. GothamLegend had to reach out to a third-party incident response team to assist with the investigation. You are a member of the IR team - all you have is an encoded Powershell script. Can you decode it and identify what malware is responsible for this attack?

## Challenge Submission

### 1. What security protocol is being used for the communication with a malicious domain? _(3 points)_

First of all I applied **Base 64** decoding and then **UTF-16LE**.

![Alt text](/assets/img/malicious-powershell-analysis/1.png)

After that we can see enough to answer question 1.

![Alt text](/assets/img/malicious-powershell-analysis/2.png)
> **TLS 12**

### 2. What directory does the obfuscated PowerShell create? (Starting from \\HOME\\) _(4 points)_

We have to decode the following string: 
```powershell
$Imd1yck=$HOME+((('UO'+'H'+'Db_')+'b'+('h3'+'0UO')+('HY'+'f')+('5be5'+'g'+'UOH'))."ReP`lACe"(('U'+'OH'),[StrInG][chAr]92))+$Swrp6tc+(('.'+'dl')+'l');$K47V=('R'+('4'+'9G'))
```

The **92 ASCII** character is `\` backslash. So we can replace `UOH` with backslash. 
The result is `HOME\db_BH30\yF5BE5\`.

>**`HOME\db_BH30\yF5BE5\`**

### 3. What file is being downloaded (full name)? _(4 points)_

Below CyberChef recipe's list to de-obfuscate malicious powershell.

![Alt text](/assets/img/malicious-powershell-analysis/3.png)

We have to focus on the following lines.

![Alt text](/assets/img/malicious-powershell-analysis/6.png)

Replace **`Swep6tc`** with **`A69S`**.

> **A69S**

#### 4. What is used to execute the downloaded file? _(3 points)_

Look for **A69S.dll** and we will find execution with **rundll32**.

![Alt text](/assets/img/malicious-powershell-analysis/5.png)

> **rundll32**

#### 5. What is the domain name of the URI ending in ‘/6F2gd/’ _(3 points)_

![Alt text](/assets/img/malicious-powershell-analysis/4.png)

> **wm.mcdevelop.net**

#### 6. Based on the analysis of the obfuscated code, what is the name of the malware? _(3 points)_

Just research on Google first part of malicious powershell script.

> **emotet**


*References*:
- [https://malware.news/t/deobfuscating-powershell-putting-the-toothpaste-back-in-the-tube/23509](https://malware.news/t/deobfuscating-powershell-putting-the-toothpaste-back-in-the-tube/23509){:target="_blank"}
- [https://www.socinvestigation.com/cooking-malicious-powershell-obfuscated-commands-with-cyberchef](https://www.socinvestigation.com/cooking-malicious-powershell-obfuscated-commands-with-cyberchef){:target="_blank"}
- [https://medium.com/mii-cybersec/malicious-powershell-deobfuscation-using-cyberchef-dfb9faff29f](https://medium.com/mii-cybersec/malicious-powershell-deobfuscation-using-cyberchef-dfb9faff29f){:target="_blank"}
- [https://gist.github.com/Neo23x0/6af876ee72b51676c82a2db8d2cd3639](https://gist.github.com/Neo23x0/6af876ee72b51676c82a2db8d2cd3639){:target="_blank"}
