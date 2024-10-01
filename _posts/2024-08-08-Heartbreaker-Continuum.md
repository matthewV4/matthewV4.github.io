---
layout: post
title: Heartbreaker-Continuum
subtitle: HTB Sherlock Writeup
gh-repo: daattali/beautiful-jekyll
tags: HackTheBox, Windows, MalwareAnalysis
comments: false
mathjax: true
---

**Sherlock Category:** Malware Analysis

**Sherlock Difficulty:** Easy

**Scenario:** Following a recent report of a data breach at their company, the client submitted a potentially malicious executable file. The file originated from a link within a phishing email received by a victim user. Your objective is to analyze the binary to determine its functionality and possible consequences it may have on their network. By analyzing the functionality and potential consequences of this binary, you can gain valuable insights into the scope of the data breach and identify if it facilitated data exfiltration. Understanding the binary’s capabilities will enable you to provide the client with a comprehensive report detailing the attack methodology, potential data at risk, and recommended mitigation steps.

# Environment Setup
Before inspecting malware, it is crucial to confirm that our environment is properly isolated to ensure safety. The image below shows the IP configuration of our VM, verifying that it is in an isolated state

![image](/assets/img/HB1.png)

After confirming that our environment is properly isolated, we can proceed to download the malicious file from HTB. We extract it to our desktop using the password “hacktheblue.”

![image](/assets/img/HB2.png)

Once extracted, we observe that the ZIP file contains an executable named Superstar_MemberCard.tiff.exe.

![image](/assets/img/HB3.png)

# Task 1
**Question:** To accurately reference and identify the suspicious binary, please provide its SHA256 hash.

I used CertUtil to obtain the SHA256 hash of the malicious executable file. CertUtil is a built-in command-line utility for Windows operating systems and can be used to get the hash of a file in PowerShell or CMD with the following command: **certutil -hashfile <PATH_TO_FILE> <HASH_ALGORITHM>**

![image](/assets/img/HB4.png)

~~~
12daa34111bb54b3dcbad42305663e44e7e6c3842f015cccbbe6564d9dfd3ea3
~~~

# Task 2
**Question:** When was the binary file originally created, according to its metadata (UTC)

Sometimes, simply checking the properties of a malicious file will yield incorrect results for its metadata, especially the creation date, as malware authors may use tools to alter timestamps to make it appear as if the file was created at a different time. This technique is often referred to as Timestomping.

We can use a tool such as **PEStudio**, which is used for spotting suspicious artifacts in executable files, to find the correct creation date of the executable file. After loading the file, we notice the creation date under the “stamps” section as March 13, 2024, 10:38:06 UTC. For reference, the image to the right displays the incorrect timestamp when simply viewing its properties through the GUI:

![image](/assets/img/HB5.png)

~~~
2024–03–13 10:38:06
~~~

# Task 3
**Question:** Examining the code size in a binary file can give indications about its functionality. Could you specify the byte size of the code in this binary?

While using PEStudio, we can navigate to the optional header (subsystem > GUI) to find the **size-of-code** header to be 38400 bytes.

![image](/assets/img/HB6.png)

~~~
38400
~~~

# Task 4
**Question:** It appears that the binary may have undergone a file conversion process. Could you determine its original filename?

We can find the original name of the file before the name change under the **resources** tab in PEStudio.

![image](/assets/img/HB7.png)

~~~
newILY.ps1
~~~

# Task 5 
**Question:** Specify the hexadecimal offset where the obfuscated code of the identified original file begins in the binary.

To solve this, I used the tool **HxD**. After opening the file and scrolling until we come across the obfuscated code segment, we find the hexadecimal offset to start at 2C74.

![image](/assets/img/HB8.png)

~~~
2C74
~~~

# Task 6 
**Question:** The threat actor concealed the plaintext script within the binary. Can you provide the encoding method used for this obfuscation?

Right away, we can tell this is Base64 encoded, hinted at by the equal signs (==). Additionally, we can also tell that we need to reverse the obfuscated script, as these equal signs usually appear at the end.

We can use a tool like **CyberChef** to reverse the script and decode the Base64 algorithm to obtain the human-readable script utilized by the executable:

Step 1: Reverse Base64 code

![image](/assets/img/HB9.png)

Step 2: Decode from Base64

![image](/assets/img/HB10.png)

~~~
Base64
~~~

# Task 7 
**Question:** What is the specific cmdlet utilized that was used to initiate file downloads?

Following the deobfuscation of the Base64 encoded code, the cmdlet Invoke-WebRequest stands out, as it can be used to download files from the web. This is evident in the image above.

~~~
Invoke-WebRequest
~~~

# Task 8
**Question:** Could you identify any possible network-related Indicators of Compromise (IoCs) after examining the code? Separate IPs by comma and in ascending order.

Using CyberChef, we can simply use the **Extract IP addresses** recipe and filter for unique IPs to find the network-related IoCs.

![image](/assets/img/HB11.png)

~~~
35.169.66.138,44.206.187.144
~~~

# Task 9
**Question:** The binary created a staging directory. Can you specify the location of this directory where the harvested files are stored?

The staging directory can be seen in the deobfuscated code being assigned to a variable named $targetDir.

![image](/assets/img/HB12.png)

~~~
C:\Users\Public\Public Files
~~~

# Task 10 
**Question:** What MITRE ID corresponds to the technique used by the malicious binary to autonomously gather data?

Analyzing the deobfuscated code, we find that the script falls under the MITRE ATT&CK technique T1119 (Automated Collection)

~~~
T1119
~~~

# Task 11
**Question:** What is the password utilized to exfiltrate the collected files through the file transfer program within the binary?

Stepping through the code, we find the password used to exfiltrate the collected files via SFTP (Secure FTP). The password can be found in the line where the SFTP connection is set up:

![image](/assets/img/HB13.png)

~~~
M8&C!i6KkmGL1-#
~~~





















