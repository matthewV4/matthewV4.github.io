---
layout: post
title: Suspicious Python Package
subtitle: LetsDefend Writeup
gh-repo: daattali/beautiful-jekyll
tags: LetsDefend, Windows, Security Analyst
comments: false
mathjax: true
---

**Role:** Security Analyst

**Difficulty:** Hard

**Scenario:** One of our employees attempted to install a Python package, and shortly afterward, someone logged into his work account. He doesn't know how it happened and needs your help as a forensics investigator to determine what occurred.

**File Location:** C:\Users\LetsDefend\Desktop\ChallengeFile\MalPy.zip

# Task 1
**Question**: The attacker downloaded a malicious package. What is the full URL?

Given a KAPE Triage, one thing that I always do first is inspect the provided Master File Table file with the MFTExplorer tool by Eric Zimmerman. The NTFS contains this $MFT which stores metadata about all files and directories on an NTFS volume. Looking at the users on the volume one that stands out the msot is the "Administrator" user. Inspecting the administrator's directories and files we come across some interesting files within the Download directory:

![image](/assets/img/SPF1.png)

Given what we know about a malicipius python package being installed to the system, we are on the right track. To find the URL that was responsbile for this malicious package, we can find it through MFTExplorer 

![image](/assets/img/SPF2.png)

# Task 2: What is the name and version of the downloaded package?
![image](/assets/img/SPF3.png)

# Task 3: What is the exact time that this package was downloaded?
![image](/assets/img/SPF4.png)

# Task 4: What file in the package contains malicious code?
![image](/assets/img/SPF5.png)
![image](/assets/img/SPF6.png)
![image](/assets/img/SPF7.png)

# Task 5: What was the name of the archive file created for exfiltration and then deleted?
![image](/assets/img/SPF8.png)

# Task 6: When did the zip file get deleted?
For this task, we can navigate to oour $J (Journal) file and filter for "temp_file.zip" to find out when it was deleted. We can parse the $J file into a CSV and inport it into Timeline Explorer doing the following:
![image](/assets/img/SPF9.png)

# Task 7: When did the zip file get deleted?

# Task 8: What exactly did the attacker steal from the victim's machine? (Name of the file)

# Task 9: The stolen file contains some sensitive data. What is the full URL of the website and the victim’s username?

# Task 10: What is the IP and PORT number of the attacker C2?

