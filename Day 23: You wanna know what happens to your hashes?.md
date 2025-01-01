In this challenge, I explored different ways to crack password-protected files and hashes, both of which are common in the world of security breaches. Passwords are often hashed before being stored in databases, but cracks in the hashing process or improper password selection can make them vulnerable. Additionally, files encrypted with passwords, such as PDFs, pose a challenge for those attempting to access sensitive data. I tackled both of these challenges in this project.

## 1. **Understanding Hash Functions**

Before diving into cracking passwords, I learned how hashes are used to store passwords safely. In the past, passwords were stored in plaintext, making them an easy target for hackers. Modern systems store passwords as hashes (e.g., SHA-256), and this way, even if an attacker gains access to the hash, they can't easily reverse-engineer it into the original password. However, the security of the hash relies on its algorithm, and older ones like MD5 or weak configurations can still be cracked.

### Hash Example:

```
d956a72c83a895cb767bb5be8dba791395021dcece002b689cf3b5bf5aaa20ac
```

I was given this hash and had to figure out what original password created it. First, I used a hash identification tool to determine that it was a SHA-256 hash. This was helpful because it told me the type of algorithm to use when cracking it.

## 2. **Using John the Ripper to Crack Password Hashes**

The next step was to crack the password hash. I used **John the Ripper**, one of the most popular password-cracking tools, to attempt to break the hash. Initially, I used a common password wordlist called **rockyou.txt**. Unfortunately, the hash didn’t match any password from this wordlist.

### The Solution: Rules and Customization

Since the password might not be a simple one from a public wordlist, I turned to John’s **rules** feature. This allows the tool to generate variations of words in the wordlist (e.g., replacing 'a' with '@', adding numbers to the end). After applying this, I successfully cracked the hash, revealing the password!

```
john --format=raw-sha256 --rules=wordlist --wordlist=/usr/share/wordlists/rockyou.txt hash1.txt
```

This method is really effective because it doesn’t just check passwords verbatim from the wordlist, but also transformations on those words. It's like testing multiple passwords at once from a single base word.

## 3. **Breaking into a Password-Protected PDF**

Next, I moved on to cracking a password-protected PDF file. The goal was to extract the password from the file so I could open it. Like the hash challenge, the first thing I did was identify the hash for the PDF using **pdf2john**. This tool generates a hash from a password-protected PDF file, just like John the Ripper can handle password hashes.

### Custom Wordlist

The PDF’s password didn’t match any common passwords, so I needed to create a custom wordlist based on what I knew about the person who created the PDF—*Mayor Malware*.

I created a wordlist with the following words:
- Fluffy
- FluffyCat
- Mayor
- Malware
- MayorMalware

These words made sense given the context. Once I had the custom wordlist, I ran John the Ripper with the **--rules=single** option to try these words and their variations.

```
john --rules=single --wordlist=wordlist.txt pdf.hash
```

This strategy worked, and I successfully cracked the PDF password. It felt great to solve the challenge using a personalized approach instead of relying on generic wordlists!

## 4. **Conclusion**

By the end of this project, I gained hands-on experience with cracking password hashes and PDF encryption. The key takeaways were:
- **Hash functions** are essential for protecting passwords, but weak or improper implementations can still be vulnerable.
- **John the Ripper** is an invaluable tool for cracking password hashes, especially when combined with wordlists and transformation rules.
- Custom **wordlists** based on the target’s context and habits can be crucial in breaking into encrypted files, such as PDFs.

The challenge was educational and gave me a deeper understanding of password protection mechanisms and how attackers exploit weak passwords or improper implementations. With this knowledge, I feel better equipped to defend against password-based attacks and help improve security practices.

**Answers**

*Crack the hash value stored in hash1.txt. What was the password?* **fluffycat12**

*What is the flag at the top of the private.pdf file?* **THM{do_not_GET_CAUGHT}**
