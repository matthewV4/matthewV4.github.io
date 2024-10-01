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
