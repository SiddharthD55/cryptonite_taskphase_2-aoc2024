## Wareville SOC Incident Investigation: Beginner’s Guide

### Setting the Scene  
In this investigation, I was tasked with analyzing a potential security incident in Wareville's network. A noisy alert indicated that multiple encoded PowerShell commands were run on various systems, triggering concerns of malicious activity. My job was to determine whether these alerts were **True Positives (TP)** (actual threats) or **False Positives (FP)** (benign activity mistaken for threats).  

I used Elastic SIEM to sift through logs, correlate events, and uncover the truth.


### Analysis  

#### 1. **Understanding the Alert**  
The alert flagged encoded PowerShell commands executed on December 1st, 2024, between 09:00 and 09:30. My first step was to narrow the log search to this specific timeframe in Elastic SIEM and add relevant fields to make the data more readable. These fields included:
- `host.hostname` (machine name),
- `user.name` (account used),
- `event.category` (type of event),
- `process.command_line` (command details), and
- `event.outcome` (success or failure).

From this, I saw that the same PowerShell command was run on multiple systems using a generic admin account (`service_admin`). Before each PowerShell execution, there was a successful login, which was suspicious.  

#### 2. **Identifying Patterns**  
I noticed the following:  
- The same user (`service_admin`) was involved in all events.  
- The commands originated from the same IP address (`10.0.11.11`).  
- The logins were extremely precise, which hinted at automated activity.

Upon inquiry, I learned that `service_admin` is a shared account used by two administrators who were not in the office during the incident. This raised more red flags.

#### 3. **Broadening the Search**  
To dig deeper, I expanded the timeframe to cover November 29th to December 1st, looking for any earlier patterns. Here’s what I found:
- **Failed Login Attempts:** There were thousands (6,791!) of failed logins over three days from `service_admin` and IP `10.0.11.11`.  
- **A Spike in Events:** A different IP address (`10.0.255.1`) appeared just before the PowerShell commands.  

#### 4. **Detecting a Brute-Force Attack**  
The failed logins suggested a brute-force attack where someone repeatedly tried to guess the account’s password. On December 1st, they succeeded:  
- At **08:54:39**, IP `10.0.255.1` successfully logged in to `ADM-01`.  
- Immediately afterward, encoded PowerShell commands were executed across systems.

#### 5. **Decoding the PowerShell Command**  
The command was Base64-encoded, so I used CyberChef to decode it. It revealed:  

```powershell
Install-WindowsUpdate -AcceptAll -AutoReboot
```  

This wasn’t malicious! The command installed Windows updates and rebooted the machines automatically. Whoever ran it wasn’t causing harm; they were fixing something.

### Key Findings  
After piecing everything together, here’s what I concluded:  
1. The failed login attempts (6,791) were part of a brute-force attack targeting `service_admin`.  
2. The attacker gained access using IP `10.0.255.1` on December 1st at 08:54:39.  
3. The attacker used their access to run PowerShell commands that fixed outdated credentials and updated systems.  
4. Surprisingly, the attacker wasn’t a threat! It was **Glitch**, someone who wanted to help secure Wareville’s defenses.  

### What I Learned  

#### 1. **True Positives vs. False Positives**  
- A **True Positive (TP)** is a legitimate threat (e.g., a brute-force attack).  
- A **False Positive (FP)** is harmless activity flagged as suspicious (e.g., Glitch's helpful actions).  

As an analyst, I learned that correctly identifying TPs vs. FPs is critical. Misclassifying TPs could allow attackers to succeed, while focusing too much on FPs wastes valuable time.

#### 2. **Correlation is Key**  
By connecting dots like IP addresses, user accounts, and event timings, I built a timeline of events. This helped me uncover the real story behind the alerts.

#### 3. **Decoding Commands**  
Encoded commands often look scary, but they’re not always malicious. Tools like CyberChef make decoding easy and can reveal the true intent.

#### 4. **Trust but Verify**  
Even “good” actions (like updating systems) should be verified. Glitch’s helpful intentions could have gone unnoticed if I hadn’t decoded the commands.

### Final Thoughts  
This investigation taught me that SOC work requires patience, attention to detail, and an open mind. Alerts might seem alarming at first glance, but with the right tools and methods, I can uncover the truth and help keep systems secure.

**Answers**
*What is the name of the account causing all the failed login attempts?* **service_admin**

*How many failed logon attempts were observed?* **6791**

*What is the IP address of Glitch?* **10.0.255.1**

*When did Glitch successfully logon to ADM-01? Format: MMM D, YYYY HH:MM:SS.SSS* **Dec 1, 2024 08:54:39.000**

*What is the decoded command executed by Glitch to fix the systems of Wareville?* **Install-WindowsUpdate -AcceptAll -AutoReboot**
