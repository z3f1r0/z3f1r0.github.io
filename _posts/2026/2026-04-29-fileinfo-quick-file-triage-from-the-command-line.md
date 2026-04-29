---
title: FileInfo - Quick File Triage from the Command Line
date: 2026-04-29
categories: [Projects]
tags: [Blog, tools, DFIR]
---

During Blue Team CTFs and DFIR activities, I often found myself needing immediate access to file hashes and essential metadata. To streamline this process, I built FileInfo — a lightweight cross-platform toolkit (Bash and PowerShell) to quickly extract key information and generate IoCs from any file.

It provides filename, full path, size, timestamps, and multiple hashes (SHA256, SHA1, MD5) in a simple way perfect for fast triage and forensic workflows.

👉 GitHub: https://github.com/z3f1r0/fileinfo

---

## Features

* Full file path resolution
* File size in bytes
* Detailed metadata
* Hashes: SHA256, SHA1, MD5
* Works on Linux and Windows

---

## Linux Usage

### Run

```bash
./fileinfo.sh /full/path/to/file
```

### Setup (optional, for global use)

```bash
chmod +x fileinfo.sh
sudo cp fileinfo.sh /usr/local/bin/fileinfo
```

Then:

```bash
fileinfo.sh /path/to/file
```

---

## Windows Usage (PowerShell)

### Run

```powershell
./Get-FileInfo.ps1 <file>
```

Example:

```powershell
./Get-FileInfo.ps1 ./sample.exe
```

### If execution is blocked

```powershell
Set-ExecutionPolicy -ExecutionPolicy Bypass -Scope Process
```

---

## Typical Use Cases

* Malware triage
* Digital forensics
* Incident response
* Threat hunting
* IoC generation

---
