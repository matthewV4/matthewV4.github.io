---
layout: post
title: Suspicious Python Package
subtitle: LetsDefend Writeup
gh-repo: daattali/beautiful-jekyll
tags: LetsDefend, Windows, Security Analyst
thumbnail-img: assets/img/PPACK.jpg
share-img: assets/img/PPACK.jpg
comments: false
mathjax: true
---
**Category:** Security Analyst

**Difficulty:** Hard

**Scenario:** One of our employees attempted to install a Python package, and shortly afterward, someone logged into his work account. He doesn't know how it happened and needs your help as a forensics investigator to determine what occurred.

# Task 1
**Question**: The attacker downloaded a malicious package. What is the full URL?

Given a KAPE Triage, one of the first things I always do is inspect the provided Master File Table (MFT) using the MFTExplorer tool by Eric Zimmerman. The NTFS file system contains the $MFT, which stores metadata about all files and directories on the volume. Upon examining the users on the volume, the one that stands out the most is the "Administrator" user. While inspecting the Administrator's directories and files, we come across some interesting files within the Downloads directory:

![image](/assets/img/SPF1.png)

Given what we know about a malicious Python package being installed on the system, we are on the right track. To identify the URL responsible for downloading this malicious package, we can locate it using MFTExplorer:

![image](/assets/img/SPF2.png)

~~~
https://github.com/0xMM0X/peloton
~~~

# Task 2: 
**Question**: What is the name and version of the downloaded package?

While navigating the "pelaton-main" folder, we come across a PKG-INFO file that contains information about the package, specifically the name and version:

![image](/assets/img/SPF3.png)

~~~
peloton-client123:0.8.10
~~~

# Task 3:
**Question**: What is the exact time that this package was downloaded?

By locating the SI_CREATED_ON timestamp, we can determine when the package was initially downloaded onto the system

![image](/assets/img/SPF4.png)

~~~
2024-01-22 20:00:11
~~~

# Task 4: 
**Question**: What file in the package contains malicious code?

Looking at the Administrator's previous ran PowerShell commands, the "setup.py" is of interest as it is being executed directly from the malicious pelaton-main  directory. 

![image](/assets/img/SPF5.png)

Opening the setp.py file we find the following: 

![image](/assets/img/SPF6.png)

Here, we see base64-encoded information, which is a red flag. We can proceed by decoding it to understand what the file is actually doing. After reversing the encoded data, decoding it from base64, and adding a recipe for Zlib Inflate, we get the following:

![image](/assets/img/SPF7.png)

~~~
setup.py
~~~

# Task 5:
**Question**: What was the name of the archive file created for exfiltration and then deleted?

![image](/assets/img/SPF8.png)

~~~
temp_file.zip
~~~

# Task 6: 
**Question**: When did the zip file get deleted?

For this task, we can navigate to our $J (Journal) file and filter for "temp_file.zip" to find out when it was deleted. We can parse the $J file into a CSV and import it into Timeline Explorer by doing the following:

![image](/assets/img/SPF9.png)

Searching for the file, we find the "FileDelete" header, which, as the name implies, tells us when a file was deleted.

![image](/assets/img/SPF10.png)

![image](/assets/img/SPF11.png)

~~~
2024-01-22 20:00:42
~~~

# Task 7: 
**Question**: What exactly did the attacker steal from the victim's machine? (Name of the file)

![image](/assets/img/SPF12.png)

~~~
Login Data
~~~

# Task 8: 
**Question**: The stolen file contains some sensitive data. What is the full URL of the website and the victim’s username?

To find out the victim's user and the URL, we can open the "Login Data" SQLite file that was exfiltrated:

![image](/assets/img/SPF13.png)

![image](/assets/img/SPF14.png)

~~~
https://app.letsdefend.io/_all4m
~~~

# Task 9: 
**Question**: What is the IP and PORT number of the attacker C2?

![image](/assets/img/SPF15.png)

~~~
172.31.78.151:8000
~~~

