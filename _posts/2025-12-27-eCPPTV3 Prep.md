---
layout: post
title: "ECPPT Certification Guide: Preparation and Methodology"
date: 2025-12-27
categories: [certifications]
tags: [eCPPTV3, penetration-testing, INE, HTB]
---

## What is eCPPT?

**ECPPT** stands for **eLearnSecurity Certified Professional Penetration Tester**.

It is a highly respected, hands-on penetration testing certification created by eLearnSecurity (eLS), now part of the **INE Security** brand since the acquisition. It's widely considered one of the best certifications for learning practical, real-world offensive security skills.

---

## Key Characteristics of the ECPPT

### 1. Performance-Based
Unlike multiple-choice exams, the ECPPT is a **practical, hands-on exam**. You are given a live network range (a lab environment) with multiple target machines, and your task is to infiltrate them, just like a real penetration test.

### 2. Comprehensive Methodology
It tests the entire penetration testing lifecycle:
- **Reconnaissance & Enumeration**
- **Vulnerability Assessment & Exploitation**
- **Post-Exploitation & Lateral Movement**
- **Privilege Escalation** (both Windows and Linux)
- **Active Directory**
- **Basic Web Penetration Testing**

### 3. Strong Focus on Fundamentals
While it covers modern techniques, it deeply emphasizes understanding core protocols (SMB, LDAP, HTTP/S, etc.), manual exploitation, and scripting (Python, PowerShell) to create your own tools.

---

## How to Prepare

I had already purchased the voucher for the exam and was planning to get premium access for the material provided by INE, as Alexis's content was excellent for eJPT. However, after researching user feedback, I found that the INE material alone was not enough to pass the exam.

Therefore, I decided not to get the premium subscription and instead prepared from other external resources.

---

## Active Directory Preparation

Since this is an AD-focused exam, here's how to prepare properly:

### Recommended Resources

- [**Introduction to Active Directory**](https://academy.hackthebox.com/module/details/74) (HTB Academy)
- [**Active Directory Enumeration & Attacks**](https://academy.hackthebox.com/module/details/143) (HTB Academy)

These should be enough for you to pwn the AD part in the exam. However, more practice is required, so consider watching some of [**IPPSec's Windows and AD videos**](https://www.youtube.com/playlist?list=PLidcsTyj9JXL4Jv6u9qi8TcUgsNoKKHNn).

---

## Tools and Techniques

### Critical Techniques
The exam relies heavily on these techniques:

- **Kerberoasting**
- **AS-REP Roasting**
- **DCSync Attacks**

**Make sure to understand these very well and practice them extensively.**

### Important Considerations

You should also expect tools to malfunction during the exam or find that tools you're used to working with are missing. Make sure to understand **Active Directory PowerShell** very well as a backup.

---

## Essential Tools

| Tool | Purpose |
|------|---------|
| **rpcclient** | Test credentials, enumerate domain users |
| **xfreerdp** | RDP to remote hosts |
| **bloodhound-python** | Collect AD data |
| **kerbrute** | Find Kerberoastable users |
| **Impacket toolkit** | Your Swiss Army knife in AD (find AS-REP Roastable users, perform DCSync attacks) |
| **CrackMapExec** | Test credentials and gain shells |
| **PowerShell** | Use `Enter-PSSession` in some cases instead of Evil-WinRM |
| **Metasploit** | Session handling |

---

## Methodology

Follow these steps for a successful approach:

1. **Enumerate valid users**
2. **Find Kerberoastable and AS-REP Roastable users**
3. **Obtain credentials and run BloodHound**
4. **Read all questions carefully and look for hints**
5. **Create custom user lists based on your findings**

### Critical Tips

- **When you land on a machine, enumerate from there, not from your attacking host.** You will find that you can RDP or use PS-Remoting from the compromised machine rather than from your attacking host.
- **Expect not to find flags where they're supposed to be** — you'll need to search for them.

---

## Password Cracking

Hash or password cracking is the **core of the exam**. Here are the wordlists you should use, **in order**:

1. `common_corporate_passwords.lst`
2. `seasons.txt`
3. `months.txt`
4. `xato-net-10-million-passwords-10000.txt`
5. `rockyou.txt`

---

## Linux and Web Application Testing

### WordPress Testing

- Complete the [**Hacking WordPress**](https://academy.hackthebox.com/module/details/17) module (HTB Academy)
- **Perform manual testing** — don't rely solely on automated tools
- Use the same wordlists mentioned above
- **Brute force everything**

---

## Final Thoughts

The exam is fun and provides excellent hands-on experience. However, in hindsight, **I wish I had invested the money into CPTS instead.**
For Active Directory techniques, see: [AD Notes](/posts/AD-Notes/)
---

**Good luck with your ECPPT journey!**

---

*If you found this guide helpful, feel free to share it with others preparing for the ECPPT certification.*
