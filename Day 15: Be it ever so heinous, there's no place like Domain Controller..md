The goal of this was to investigate a potential breach on an Active Directory domain controller, and by doing so, learn about common Active Directory attacks and how to respond to them. Active Directory (AD) is a crucial part of enterprise networks, managing resources and user access within a domain. Understanding how attackers can compromise AD and recognizing key signs of a breach is essential for defending against these attacks.

### Setup and Connection

Before diving into the investigation, I started by setting up the machine and connecting to it through RDP. The credentials were straightforward, provided in the connection card, and once I was in, I was able to access the domain controller where the breach was suspected. The domain controller was where all the AD data was stored, and it was vital to inspect it thoroughly.

I began by reviewing the components of Active Directory: domains, domain controllers (DC), users, groups, organizational units (OUs), and Group Policy Objects (GPOs). These are the building blocks of AD that an attacker could potentially exploit.

### Investigating the GPOs

The first thing I did was audit the Group Policy Objects (GPOs) on the domain controller. GPOs are a great way to enforce security policies and configurations across the domain, but if they’re misconfigured or manipulated, they can serve as a vector for an attacker to spread malware or malicious configurations across the entire network.

Using PowerShell, I ran the command `Get-GPO -All` to list all GPOs in the domain. It was crucial to check for any suspicious or out-of-place GPOs, as these could be signs that an attacker had set them up. For example, I looked for unusual GPOs that had recently been modified.

```powershell
PS C:\Users\Administrator> Get-GPO -All | Where-Object { $_.ModificationTime } | Select-Object DisplayName, ModificationTime
```

This PowerShell command filters for recently modified GPOs, which is useful when looking for signs of a breach. When I reviewed the modification times, I found an interesting one called **SetWallpaper** that had been altered recently. This caught my attention because, while setting a desktop wallpaper seems harmless, attackers sometimes use GPOs to deploy malicious scripts, software, or even backdoors. 

I exported the GPO to an HTML file for closer inspection with the `Get-GPOReport` cmdlet:

```powershell
PS C:\Users\Administrator> Get-GPOReport -Name "SetWallpaper" -ReportType HTML -Path ".\SetWallpaper.html"
```

Upon reviewing the HTML report, I saw that the GPO was set to deploy a wallpaper with the image located at `C:\THM.jpg` on the domain controller. While this could be innocent, it was possible the image could contain some form of payload or script designed to execute when the wallpaper was deployed.

### Reviewing Event Viewer Logs

Next, I turned to the **Event Viewer** to check for any suspicious activity. The Event Viewer logs user logins, security events, and service behaviors that could provide insight into what’s happening on the domain. I focused on the **Security** logs and looked for unusual login activity, failed login attempts, or signs of privilege escalation.

A specific event ID that stood out to me was **4768**, which logs when a TGT (Ticket Granting Ticket) request is made for a high-privilege account. I paid close attention to this because if an attacker had escalated their privileges, they might be requesting TGTs for sensitive accounts like Domain Admins.

```powershell
Event ID 4768: A TGT (Kerberos) ticket was requested for a high-privileged account.
```

These types of events could indicate Kerberos attacks, like **Kerberoasting**, where attackers attempt to crack service account passwords by requesting service tickets. If I found multiple such events in a short period, it would be a sign that further investigation was needed.

### Auditing User Accounts

After checking GPOs and logs, I decided to take a deeper look at the user accounts in the domain. I used the **Search-ADAccount** cmdlet to search for locked-out accounts, as password spraying or brute-force attacks can trigger account lockouts. Here’s the command I used:

```powershell
Search-ADAccount -LockedOut | Select-Object Name, SamAccountName, LockedOut, LastLogonDate, DistinguishedName
```

I also wanted to see what groups users belonged to, as high-privilege groups like **Domain Admins** or **Enterprise Admins** are prime targets for attackers. Running the following PowerShell command helped me quickly list users and their group memberships:

```powershell
PS C:\Users\Administrator> Get-ADUser -Filter * -Properties MemberOf | Select-Object Name, SamAccountName, @{Name="Groups";Expression={$_.MemberOf}}
```

Looking through the list, I could spot a few users with suspicious group memberships, like **cmnatic**, who was in the **Domain Admins** group. This raised red flags because the attacker could have been trying to escalate their privileges or use a compromised admin account to gain control of the domain.

### PowerShell History and Logs

One of the final things I checked was the PowerShell command history. This history can be invaluable for understanding what actions an attacker may have taken on the system. The PowerShell history is stored in a file located at `%APPDATA%\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt`. I opened this file to review the commands that had been executed, which could give clues as to what steps the attacker had taken.

Additionally, I looked into PowerShell logs in the Event Viewer, specifically under **Microsoft > Windows > PowerShell > Operational**. This log contained detailed records of all PowerShell scripts and commands executed on the system.

### Conclusion

So, I was able to identify potential signs of a breach in the Active Directory environment. The suspicious GPO modification, unusual log entries, and user account anomalies indicated that something malicious might have been happening. I concluded that the breach involved an attacker who had likely gained high-level privileges within the domain, potentially exploiting Kerberos authentication weaknesses or abusing GPOs to maintain persistence.

This task reinforced the importance of regularly auditing GPOs, user activity, and reviewing system logs in the event of a suspected breach. By understanding how Active Directory works and the common methods attackers use to exploit it, I feel more confident in my ability to detect and respond to future security incidents.

**Answers**

*On what day was Glitch_Malware last logged in? Answer format: DD/MM/YYYY* **07/11/2024**
 
*What event ID shows the login of the Glitch_Malware user?* **4624**
 
*Read the PowerShell history of the Administrator account. What was the command that was used to enumerate Active Directory users?* **Get-ADUser -Filter * -Properties MemberOf | Select-Object Name**
 
*Look in the PowerShell log file located in Application and Services Logs -> Windows PowerShell. What was Glitch_Malware's set password?* **SuperSecretP@ssw0rd!**
 
*Review the Group Policy Objects present on the machine. What is the name of the installed GPO?* **Malicious GPO - Glitch_Malware Persistence**
