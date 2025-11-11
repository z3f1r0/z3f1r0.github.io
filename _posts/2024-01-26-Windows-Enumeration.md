---
title: Windows Enumeration
date: 2024-01-26
categories: [Blog]
tags: [Windows, Enumeration]
---

The following list of commands can be used in a local Windows environment to enumerate all useful information.

## Windows Operating System Enumeration

### Basic OS Information

```powershell
Get-ComputerInfo
```
#### Installed Patches

```powershell
Get-CimInstance -query 'select * from win32_quickfixengineering' | foreach $_.hotfixid {Get-Hotfix}
```

Use the attribute `-description "Security update"` of `Get-Hotfix` to list only security updates.

```
wmic qfe get Caption,Description,HotFixID,InstalledOn
```

#### Installed Drivers (requires elevated privileges)

```powershell
Get-WindowsDriver -Online -All
```

### CPU Version and Architecture

```
systeminfo | findstr /B /C:"OS Name" /C:"OS Version" /C:"System Type"
```

```powershell
[System.Environment]::OSVersion
```

```
wmic os get OSArchitecture
```

### Installed Software

```powershell
Get-CimInstance -namespace "root/cimv2" -ClassName Win32_Product
```

#### User Software

```powershell
Get-ChildItem 'C:\Program Files', 'C:\Program Files (x86)'
```

### File System

#### Desktop Data

```powershell
Get-ChildItem 'C:\Users\<USER>\Desktop\'
```

#### User Data and Documents

```powershell
Get-ChildItem 'C:\Users\<USER>\Documents\' 
```

#### '.lnk' Files

```powershell
Get-ChildItem 'C:\Users\<USER>\AppData\Roaming\Microsoft\Windows\Recent\*'
```
#### Shared Folders

```powershell
Get-CimInstance -namespace "root/cimv2" -ClassName Win32_Share
```
#### Volumes

```powershell
Get-Partition
```

#### Environment Variables

```powershell
Get-ChildItem Env:
```

### Sources

- [https://nored0x.github.io/red-teaming/windows-enumeration/#operating-system](https://nored0x.github.io/red-teaming/windows-enumeration/#operating-system)
- [https://github.com/zweilosec/Infosec-Notes/blob/master/windows-1/windows-redteam/enumeration.md](https://github.com/zweilosec/Infosec-Notes/blob/master/windows-1/windows-redteam/enumeration.md)

## Network Enumeration

### Network Enumeration via cmd

#### Network Discovery

```
net view /all

net view \\<HOST NAME>
```

##### Basic ping scan and write output to a file

```
for /L %i in (1,1,254) do ping -w 30 -n 1 192.168.1.%i | find "Reply" >> <OUTPUT FILE NAME>.txt
```

#### Network Interfaces

```
ipconfig /all
```

#### Active Connections

```
netstat -ano
```

#### Routing Table

```
netstat -r
```

#### Hosts File

```
type %SYSTEMROOT%\system32\drivers\etc\hosts
```

#### ARP Cache

```
arp -a
```

#### NETBIOS

##### Basic nbtstat scan

```
nbtstat -A <IP ADDRESS>
```

#### Cached NetBIOS info on localhost

```
nbtstat -c
```

##### Script loop scan

```
for /L %i in (1,2,254) do nbstat -An 192.168.1.%i
```
### Network Enumeration via Powershell

#### Connection Profile

```powershell
Get-NetConnectionProfile 
```

#### Network Interfaces

```powershell
Get-NetAdapter
```

#### Routes

```powershell
Get-NetRoute
```

#### Active Connections

```powershell
Get-NetTCPConnection
```

#### ARP Table

```powershell
Get-NetNeighbor
```

## Users and Groups Enumeration

### Users Information

```powershell
Get-LocalUser | Select *
```

```
net user <USER>
```
### Local Users

```powershell
Get-LocalUser | Format-Table Name,Enabled,LastLogon,SID
```

```powershell
Get-CimInstance -class Win32_UserAccount
```

```
net users
```

### Users Home Folder List

```powershell
Get-ChildItem 'HKLM:\Software\Microsoft\Windows NT\CurrentVersion\ProfileList' | ForEach-Object { $_.GetValue('ProfileImagePath') }
```
### Local Groups

```powershell
Get-LocalGroup | Format-Table Name,SID,Description
```

```powershell
net localgroup
```

### Members of the Administrators Group

```powershell
Get-LocalGroupMember Administrators | Format-Table Name,PrincipalSource,SID
```

```powershell
net localgroup Administrators
```

### Display Who is Currently Logged In

```
qwinsta
```

#### Sources
- [https://github.com/tom0li/collection-document/blob/master/Blue%20Team%20Field%20Manual.pdf](https://github.com/tom0li/collection-document/blob/master/Blue%20Team%20Field%20Manual.pdf)

## Services Enumeration

### Services Enumeration via Powershell or cmd

#### Running Services

```powershell
Get-Service | Where-Object {$_.Status -eq 'Running'}
```

List of all services with their ProcessID for those running (cmd.exe): `wmic service list brief`

#### Unquoted Service Paths

```powershell
Get-WmiObject -class Win32_Service -Property Name, DisplayName, PathName, StartMode | Where {$_.PathName -notlike "C:\Windows*" -and $_.PathName -notlike '"*'} | select Name,DisplayName,StartMode,PathName
```

```
wmic service get name,displayname,pathname,startmode |findstr /i "auto" |findstr /i /v "c:\windows\\" |findstr /i /v """
```

Another way to enumerate unquoted service paths is through the use of the tool [winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS).

#### Sources

- [https://juggernaut-sec.com/unquoted-service-paths/](https://juggernaut-sec.com/unquoted-service-paths/)
- [https://www.ired.team/offensive-security/privilege-escalation/unquoted-service-paths](https://www.ired.team/offensive-security/privilege-escalation/unquoted-service-paths)
- [https://github.com/zweilosec/Infosec-Notes/blob/master/windows-1/windows-redteam/enumeration.md](https://github.com/zweilosec/Infosec-Notes/blob/master/windows-1/windows-redteam/enumeration.md)

## Security Systems Enumeration

### Security Systems Enumeration via cmd and Powershell

#### Antivirus Enumeration

```
wmic /namespace:\\root\securitycenter2 path antivirusproduct
```
#### AV Software Enumeration with Powershell

```powershell
Get-CimInstance -Namespace root/SecurityCenter2 -ClassName AntivirusProduct
```

#### Checking Windows Defender Service Status

```powershell
Get-Service WinDefend
```

#### Current Status of Security Tools, including Anti-Spyware, Antivirus, LoavProtection, Real-time protection, etc.

```powershell
Get-MpComputerStatus
````

#### Check Real-Time Protection

```powershell
Get-MpComputerStatus | select RealTimeProtectionEnabled
````

#### Check if the Firewall is Enabled

```powershell
Get-NetFirewallProfile -All | Format-Table Name, Enabled
```
#### Firewall Rules Enumeration

```powershell
Get-NetFirewallRule | select DisplayName, Enabled, Description
```

#### Disable Firewall (requires admin privileges)

```powershell
Set-NetFirewallProfile -Profile Domain, Public, Private -Enabled False
```

#### Check if an Incoming Connection on Port 80 is Open and Allowed in the Firewall

```powershell
Test-NetConnection -ComputerName 127.0.0.1 -Port 80
```

#### Details of Threats Detected by MS Defender

```powershell
Get-MpThreat
```

#### Check if there is a Process Named "Sysmon"

```powershell
Get-Process | Where-Object { $_.ProcessName -eq "Sysmon" }
```

#### Check Sysmon Services

```powershell
Get-CimInstance win32_service -Filter "Description = 'System Monitor service'"` or `Get-Service | where-object {$_.DisplayName -like "*sysm*"}
```

#### Check Windows Event Log for Sysmon

```
reg query HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\WINEVT\Channels\Microsoft-Windows-Sysmon/Operational
```

#### Search for Sysmon Configuration File

```
findstr /si '<ProcessCreate onmatch="exclude">' C:\tools\*
```

#### Sources:

- [https://csbygb.gitbook.io/pentips/red-team/av-enumeration](https://csbygb.gitbook.io/pentips/red-team/av-enumeration)

### Security Systems Enumeration via Scripts

#### **[Invoke-EDRChecker](https://github.com/PwnDexter/Invoke-EDRChecker)**

Enumerates the target host by querying running processes, process metadata, DLLs loaded in the current process and every metadata dlls, known installation paths, installed services, registry, and running drivers, then checks the output against a list of known defensive products such as AV, EDR, and logging tools.

```
_EXAMPLE
PS C:\> Invoke-EDRChecker
PS C:\> Invoke-EDRChecker -Force
PS C:\> Invoke-EDRChecker -Remote <hostname>
PS C:\> Invoke-EDRChecker -Remote <hostname> -Ignore
```

#### Sources:

- [https://github.com/PwnDexter/Invoke-EDRChecker/tree/master](https://github.com/PwnDexter/Invoke-EDRChecker/tree/master)

#### **[SharpEDRChecker](https://github.com/PwnDexter/SharpEDRChecker)**

A new and improved C# implementation of Invoke-EDRChecker. Checks running processes, process metadata, DLLs loaded in the current process and the metadata of each DLL, common installation directories, installed services and the metadata of each service binary, installed drivers and the metadata of each driver, all for the presence of known defensive products like AV, EDR, and logging tools. Catches even hidden EDRs through its metadata checks, more information can be found in [this blog post](https://redteaming.co.uk/2021/03/18/sharpedrchecker/).

```
.\SharpEDRChecker.exe
run-exe SharpEDRChecker.Program SharpEDRChecker
```

#### Sources:

- [https://github.com/PwnDexter/SharpEDRChecker](https://github.com/PwnDexter/SharpEDRChecker)

## Windows System Enumeration via Tools

### **[winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)**

[winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS) is a tool included in the PEASS-ng (Privilege Escalation Awesome Scripts SUITE next generation) repository on GitHub. It is designed for privilege escalation and system enumeration in Windows environments. The tool offers a comprehensive set of checks and gathers valuable system information to identify potential vulnerabilities.

In terms of enumeration, winPEAS can perform a wide variety of checks on different aspects of the system, including:

- **System and configuration information**: Details about the operating system, network configuration, installed patches, running services, etc.
- **Credentials**: Searching for stored credentials, configuration files with possible credentials.
- **User permissions and rights**: Enumeration of users, their groups, file permissions, and security policies.
- **Scheduled tasks and auto-start applications**: Identifying tasks that could be exploited for privilege escalation.
- **Common vulnerabilities**: Checking for misconfigurations, known vulnerabilities, and missing patches.
- **File and directory information**: Enumeration of files and directories with improper permissions or suspicious configurations.

Using winPEAS can provide a detailed view of the security status of a Windows system, highlighting areas that might be vulnerable or misconfigured.

#### Sources

- [https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS](https://github.com/carlospolop/PEASS-ng/tree/master/winPEAS)
- [https://www.101labs.net/comptia-security/lab-85-how-to-enumerate-for-privilege-escalation-on-a-windows-target-with-winpeas/#:~:text=WinPEAS%20is%20a%20script%20which,escalate%20privileges%20on%20Windows%20hosts.&text=Lab%20Topology%3A,Windows%20machine%20for%20this%20lab.](https://www.101labs.net/comptia-security/lab-85-how-to-enumerate-for-privilege-escalation-on-a-windows-target-with-winpeas/#:~:text=WinPEAS%20is%20a%20script%20which,escalate%20privileges%20on%20Windows%20hosts.&text=Lab%20Topology%3A,Windows%20machine%20for%20this%20lab.)
- [https://medium.com/@s12deff/windows-privilege-escalation-with-winpeas-94be6fb0f173](https://medium.com/@s12deff/windows-privilege-escalation-with-winpeas-94be6fb0f173)
- [https://www.youtube.com/watch?v=dSa_mdg3gCg](https://www.youtube.com/watch?v=dSa_mdg3gCg)
