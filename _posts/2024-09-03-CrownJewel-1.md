---
layout: post
title: CrownJewel-1
subtitle: HTB Sherlock Writeup
gh-repo: daattali/beautiful-jekyll
tags: HackTheBox, Windows, DFIR
comments: false
mathjax: true
---

**Sherlock Category:** DFIR

**Sherlock Difficulty:** Very Easy

**Scenario:** Forela’s domain controller is under attack. The Domain Administrator account is believed to be compromised, and it is suspected that the threat actor dumped the NTDS.dit database on the DC. We just received an alert of vssadmin being used on the DC, since this is not part of the routine schedule we have good reason to believe that the attacker abused this LOLBIN utility to get the Domain environment’s crown jewel. Perform some analysis on provided artifacts for a quick triage and if possible kick the attacker as early as possible.

# Task 1
**Question:** Attackers can abuse the vssadmin utility to create volume shadow snapshots and then extract sensitive files like NTDS.dit to bypass security mechanisms. Identify the time when the Volume Shadow Copy service entered a running state.

In our SYSTEM event log, we can filter for Sysmon event ID 7036 (service has entered the running or stopped state) and search for “Volume Shadow Copy.”

![image](/assets/img/)

~~~
2024–05–14 03:42:16
~~~

# Task 2
**Question:** When a volume shadow snapshot is created, the Volume shadow copy service validates the privileges using the Machine account and enumerates User groups. Find the User groups it enumerates, the Subject Account name, and also identify the Process ID (in decimal) of the Volume shadow copy service process

Shifting to the Security event logs, we can filter for event code 4799 (A user’s local group membership was enumerated / A security-enabled local group membership was enumerated.). Sifting through the logs, we come across some user groups being enumerated with the VSSVC.exe process, which is the Windows service that manages Volume Shadow Copies. It is with these logs in which we can find that the Administrators and Backup Operators groups are being enumerated on DC01$

![image](/assets/img/)

~~~
Administrators, Backup Operators, DC01$
~~~

# Task 3
**Question:** Identify the Process ID (in Decimal) of the volume shadow copy service process.

In the previous images, we notice the “Process ID” of 0x1190 in hexadecimal format. You can convert this to decimal in a variety of different ways, but I used a command-line utility called hex2dec from Sysinternals:

![image](/assets/img/)

~~~
4496
~~~

# Task 4
**Question:** Find the assigned Volume ID/GUID value to the Shadow copy snapshot when it was mounted

To find out this information, we can navigate to our NTFS log file. NTFS, or the New Technology File System, is the file system that Windows operating systems use for storing and retrieving files on hard disks and solid-state drives. We can filter for event code 4 (The NTFS volume has been successfully mounted) and do our search at the timestamp we previously found in the first question to find the Volume ID of the mounted shadow copy:

![image](/assets/img/)

~~~
{06c4a997-cca8–11ed-a90f-000c295644f9}
~~~

# Task 5
**Question:** Identify the full path of the dumped NTDS database on disk.

Utilizing the MFT file provided to us, we can open it up in MFT Explorer by Eric Zimmerman. Exploring the Users and in particular the Administrator, we find two files, ntds.dit and SYSTEM under the path: C:\Users\Administrator\Documents\backup_sync_Dc

![image](/assets/img/)

~~~
C:\Users\Administrator\Documents\backup_sync_Dc\Ntds.dit
~~~

# Task 6
**Question:** When was newly dumped ntds.dit created on disk?

In the “Overview” pane on the bottom right, we can see just when the ntds.dit was created on the disk:

![image](/assets/img/)

~~~
2024–05–14 03:44:22
~~~

# Task 7
**Question:** A registry hive was also dumped alongside the NTDS database. Which registry hive was dumped and what is its file size in bytes?

While under the same folder path, we notice a SYSTEM hive, which was also dumped alongside the NTDS.DIT. We also notice in the overview pane that it has a “physical size” of 0x10C0000:

![image](/assets/img/)

As done previously, we can run the hex through a tool to find what it would be in decimal, which would give us the total bytes of the registry hive:

![image](/assets/img/)

~~~
SYSTEM, 17563648
~~~


