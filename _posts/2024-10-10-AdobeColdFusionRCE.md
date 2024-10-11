---
layout: post
title: Adobe ColdFusion RCE
subtitle: LetsDefend Writeup
gh-repo: daattali/beautiful-jekyll
tags: LetsDefend, Windows, Security Analyst
comments: false
mathjax: true
---
**Category:** Security Analyst

**Difficulty:** Medium

**Scenario:** Our EDR software was triggered, alerted, and isolated a web server for suspicious use of the “nltest.exe” command. Investigate the Windows Event logs to determine what occurred.

# Task 1
**Question**: 

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~
https://webhook.site/33c1dca9-b63a-470d-8d6d-cb46b76932ae
~~~

# Task 2: 
**Question**: What was the IP of the third-party service to determine if the server is susceptible to remote code execution?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~
46.4.105.116
~~~

# Task 3: 
**Question**: The attacker drops a web shell backdoor. What is the web shell backdoor script written in?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~
ColdFusion Markup Language
~~~

# Task 4: 
**Question**: What is the attacker’s working directory when injecting the web shell script?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~
C:\Fusion21\cfusion\bin\
~~~

# Task 5: 
**Question**: The web shell was saved to what file?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~
..\wwwroot\cf_scripts\cfclient\huqVgdoFd.cfm
~~~

# Task 6: 
**Question**: What is the full execution command string in the web shell backdoor?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~

~~~

# Task 7: 
**Question**: The Attacker creates a reverse shell with PowerShell. What is the IP and port number the reverse shell calls back to?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~
185.100.233.201:80
~~~

# Task 8: 
**Question**: Based on the details of the log, what underlining software is the website coded in?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~

~~~

# Task 9: 
**Question**: Based on the software the website runs on and the date of the intrusion, what CVE was the attacker likely using to get remote code execution on the server?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/RCE1.png)

~~~
CVE-2023-26360
~~~






