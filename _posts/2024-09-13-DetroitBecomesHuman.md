---
layout: post
title: Detroit Becomes Human
subtitle: HTB Sherlock Writeup
gh-repo: daattali/beautiful-jekyll
tags: HackTheBox, Windows, DFIR
comments: false
mathjax: true
---

**Sherlock Category:** DFIR

**Sherlock Difficulty:** Hard

**Scenario:** Alonzo Spire is fascinated by AI after noticing the recent uptick in usage of AI tools to help aid in daily tasks. He came across a sponsored post on social media about an AI tool by Google. The post had a massive reach, and the Page which posted had 200k + followers. Without any second thought, he downloaded the tool provided via the Post. But after installing it he could not find the tool on his system which raised his suspicions. A DFIR analyst was notified of a possible incident on Forela’s sysadmin machine. You are tasked to help the analyst in analysis to find the true source of this odd incident.

# Task 1
**Question:** What is the full link of a social media post which is part of the malware campaign, and was unknowingly opened by Alonzo spire?

We are provided with a Kroll Artifact Parser and Extractor (KAPE) triage, which we can use to retrieve forensic data. We know the malware was spread through social media links, so our first step is to focus on browsing history data files. In the triage file, we navigate to detroitbecomehuman\Triage\C\Users\alonzo.spire\AppData\Local\Microsoft\Edge\User Data\Default and find the “History” SQLite database file, which contains a record of the user’s browsing history on Microsoft Edge. Filtering through the visited URLs, we find a specific social media post that is of interest:

![image](/assets/img/detroit1.png)

~~~
https://www.facebook.com/AI.ultra.new/posts/pfbid0BqpxXypMtY5dWGy2GDfpRD4cQRppdNEC9SSa72FmPVKqik9iWNa2mRkpx9xziAS1l
~~~

# Task 2
**Question:** Can you confirm the timestamp in UTC when alonzo visited this post?

Along with this, we find a very long integer, 13355296200136503, which represents our timestamp of when Alonzo visited the malicious link.

![image](/assets/img/detroit2.png)

![image](/assets/img/detroit3.png)

~~~
2024–03–19 04:30:00
~~~

# Task 3
**Question:** Alonzo downloaded a file on the system thinking it was an AI Assistant tool. What is name of the archive file downloaded?

Continuing with our investigation into the “History” SQLite database file, we notice it contains a tabled labeled “downloads.”

![image](/assets/img/detroit4.png)

~~~
AI.Gemini Ultra For PC V1.0.1.rar
~~~

# Task 4
**Question:** What was the full direct url from where the file was downloaded?

For finding the exact URL from which the file was downloaded, we can examine the “downloads_url_chains” table. This table stores the complete sequence of URLs involved in a download, including any redirects and the final URL where the file was retrieved.

![image](/assets/img/detroit5.png)

~~~
https://drive.usercontent.google.com/download?id=1z-SGnYJCPE0HA_Faz6N7mD5qf0E-A76H&export=download
~~~

# Task 5
**Question:** Alonzo then proceeded to install the newly download app, thinking that its a legit AI tool. What is the true product version which was installed?
# Task 6
**Question:** When was the malicious product/package successfully installed on the system?

Both of these tasks can be solved by transitioning over to the windows event logs provided to us, specifically, the “Application” log. The Application log can be helpful for tracking software installation events.

Filtering for the term “Gemini,” we identified an MsiInstaller Event (ID 1042), which indicates the completion of an MSI installation process. This event helps us determine the exact time the package was successfully installed on the system.

![image](/assets/img/detroit6.png)

By analyzing the subsequent MsiInstaller Event (ID 1033), we gathered additional details, such as the product name and version of the installed software, revealing it to be version 3.32.3.

![image](/assets/img/detroit7.png)

~~~
Task 5 Solution: 3.32.3
Task 6 Solution: 2024–03–19 04:31:33
~~~

# Task 7
**Question:** The malware used a legitimate location to stage its file on the endpoint. Can you find out the Directory path of this location?

To investigate further, we can turn to the PowerShell logs provided to us to find any malicious scripts or commands being used. Examining the logs within the relevant time frame, we immediately notice a red flag:

![image](/assets/img/detroit8.png)

The file path ‘C:\Program Files (x86)\Google\Install\’ raises suspicion as the Program Files (x86) directory is typically reversed for legitimate program installations. Essentially, by staging the ru.ps1 script inside of this directory, the attacker is attempting to evade detection.

~~~
C:\Program Files (x86)\Google
~~~

# Task 8
**Question:** The malware executed a command from a file. What is name of this file?

Knowing the staging directory path, we can further investigate by loading the provided $MFT file into MFT Explorer. By navigating through the MFT records in the known staging directory (C:\Program Files (x86)\Google\Install), we can locate the file responsible for executing the malware's command.

![image](/assets/img/detroit9.png)

~~~
install.cmd
~~~

# Task 9
**Question:** The malware executed a command from a file. What is name of this file?

By examining the install.cmd file located in the staging directory, we can retrieve the file contents in ASCII representation as displayed in the MFT Explorer output. These contents, shown in the image above, reveal the exact command executed by the malware.

~~~
@echooffpowershell-ExecutionPolicyBypass-File”%~dp0nmmhkkegccagdldgiimedpic/ru.ps1"
~~~

# Task 10
**Question:** What was the command executed from this file according to the logs?

The command executed from the file — according to the logs — was previously found in the Powershell logs from task #7.

~~~
powershell -ExecutionPolicy Bypass -File C:\Program Files (x86)\Google\Install\nmmhkkegccagdldgiimedpic/ru.ps1
~~~

# Task 11
**Question:** Under malware staging Directory, a js file resides which is very small in size. What is the hex offset for this file on the filesystem?

In the staging directory, we can identify the small-sized .js file and can find the hex offset in the details section in the bottom panel.

![image](/assets/img/detroit10.png)


~~~
3E90C00
~~~

# Task 12
**Question:** Recover the contents of this js file so we can forward this to our RE/MA team for further analysis and understanding of this infection chain. To sanitize the payload, remove whitespaces.

Once again, in the Details panel, we can find the ASCII representation of the file, which provides crucial information. This data can be forwarded to the RE/MA team for further analysis.

![image](/assets/img/detroit11.png)

~~~
varisContentScriptExecuted=localStorage.getItem(‘contentScriptExecuted’);if(!isContentScriptExecuted){chrome.runtime.sendMessage({action:’executeFunction’},function(response){localStorage.setItem(‘contentscriptExecuted’,true);});}
~~~


# Task 13
**Question:** Upon seeing no AI Assistant app being run, alonzo tried searching it from file explorer. What keywords did he use to search?
# Task 14
**Question:** When did alonzo searched it?

To filter through what the user may have searched in File Explorer, we can give RegRipper a try. RegRipper is a tool for parsing Windows registry files, and, in this case, it can be used to retrieve search queries made by the user in File Explorer. Specifically, we can leverage the user’s NTUSER.DAT file, which stores user-specific registry settings, and the WordWheelQuery plugin, to extract the most recent search terms entered by the user.

![image](/assets/img/detroit12.png)

~~~
Task 13 Solution: Google Ai Gemini tool
Task 14 Solution: 2024–03–19 04:32:11
~~~

# Task 15
**Question:** After alonzo could not find any AI tool on the system, he became suspicious, contacted the security team and deleted the downloaded file. When was the file deleted by alonzo?

To determine when the AI Gemini file was deleted, we can navigate to the $Recycle.Bin folder in MFT Explorer. While exploring the entries in the bin, we found an entry that clearly references the AI Gemini file within its hex data. This confirms that the deleted file was indeed the malicious AI Gemini file. By looking at the metadata in the MFT Explorer, such as timestamps, we were able to determine when the file was deleted by Alonzo

![image](/assets/img/detroit13.png)

~~~
2024–03–19 04:34:16
~~~

# Task 16
**Question:** Looking back at the starting point of this infection, please find the md5 hash of the malicious installer.

~~~
BF17D7F8DAC7DF58B37582CEC39E609D
~~~









