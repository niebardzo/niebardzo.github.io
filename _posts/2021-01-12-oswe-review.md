---
layout: post
title: AWAE and OSWE review
subtitle: 
gh-repo: daattali/beautiful-jekyll
tags: [offsec, hacking, appsec]
comments: true
---

At the end of the 2020, I took the Advanced Web Application Exploitation (AWAE) course by Offensive Security. After the course, at the beginning of 2021 I have successfully passed the Offensive Security Web Expert (OSWE) exam on the first attempt. This blog post is written to share my path, and point of view on the OSWE certification.

## Context

At the point when I started the AWAE course, I was working as the Web Application and Cloud Penetration Tester in a big financial company. As a part of my training budget, I have been bought 90 days of AWAE lab time. Compare with PWK/OSCP where I have rooted 35 machines (which was enough to pass the OSCP exam on the first attempt), in AWAE I have been able to complete all learning materials with all extra miles in 90 days without rashing too much. If you pay for the course from your pocket, I recommend buying 60 days of AWAE lab time, especially when you have bug bounty/penetration testing experience and you can read other's people code.

## AWAE Course

Unlike, the PWK the AWAE course is heavily focused on the description and analysis of real vulnerabilities in real web applications. The Offsec team teaches you on the example by showing you how to find the vulnerable place and trace the code execution to find the user-controlled input. The whole curriculum can be found on the following page:
[https://www.offensive-security.com/awae-oswe/](https://www.offensive-security.com/awae-oswe/)

The new things that I have learned at the course, that are not normally done in the day to day job as the penetration tester were:
- Exploitation of XSS, more advanced than just running alert(1),
- Manual exploitation of SQL injection,
- PHP Type Juggling,
- How to find gadgets for the deserialization exploitation
- Attacking low entropy - pseudorandomness generators
- Bypassing JS sandboxes.

It is because as the penetration tester you are mainly focused on finding the bugs and for the impact demonstration (exploitation), you can then use the automated tool or do the simplest thing e.g. in XSS you send a single XHR request to perform the money transfer. In other cases, some types of bugs are not very likely to occur in financial applications like PHP Type Juggling. 

To successfully pass the OSWE exam, you would have to obtain the following skills from the course:
- Know common vectors for authentication bypass and remote code execution in various technologies e.g. Auth Bypass in PHP can be done by Type Juggling, SQL injection..., remote code execution could be done by Unrestricted File Upload, SSTI...
- Being able to analyze the code of the application, know-how the common web frameworks structure their application e.g. MVC. (Use IDE, text editor, or just grep for the function names or pattern which may lead to hunted vulnerabilities)
- Decompiling, debugging and browsing the logs of web applications and databases.
- Scripting of chained PoC exploit in your favorite scripting language. 
- Develop a methodology for approaching white-box penetration testing of a web application. ( Imagine you on the exam, you have been given the debugging machine with web app running on it, what do you do first? what do you do next?)
- Documenting your thinking process.

My advice is to think about those skills as the goal of completing the AWAE course. The final OSWE exam would be proof that you have successfully obtained those skills, nothing else. Whenever you feel missing in some of the areas, you should go back to the course and complete some extra miles yourself by scripting the whole PoC. 

## OSWE exam

The whole proctoring exam for 48h, was a little bit stressful for me. The good thing is that when you want to do the longer break, you can switch off the proctoring session - your VPN access would be suspended for this period, and then come back to it later.

Few tips which you can find useful:
- For the PoCs modify the scripts you have written over the AWAE course. Do not write them from scratch.
- Document the vulnerability discovery (code analysis) and exploitation as you go. I have built the practice of writing the report along with the testing - which helps me to arrange my thoughts.
- Research potential vectors - use Google for that. If you find the potential authentication bypass, try to research the vulnerable function to confirm the vulnerability.

For me, it took around 30 hours to complete the exam with the report written. As I have completed everything and documented the process very thoroughly, I have received the results from Offsec the next day. 

![OSWE-passed](https://github.com/niebardzo/niebardzo.github.io/raw/master/img/2021-01-12-oswe.png)



