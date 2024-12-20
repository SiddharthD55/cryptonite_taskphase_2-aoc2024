### **The Scenario**

The town of Wareville is preparing for a big day — **G-Day**, where all the elves deliver gifts to the townsfolk. The website that schedules these gifts is called **GiftScheduler**, and it plays a crucial role in making sure everything goes smoothly. However, the website uses a **self-signed certificate**, which is a huge security issue.

Self-signed certificates are not trusted by browsers, and that’s where my plan to carry out a **man-in-the-middle attack** (MITM) comes into play. Self-signed certificates can be exploited if a hacker is able to intercept the traffic between the client and the server, and that’s exactly what I was planning to do. By using Burp Suite, I would be able to listen in on the communication, capture sensitive data like credentials, and potentially disrupt the entire G-Day operation. 

### **Understanding the Basics: Self-Signed Certificates**

Before diving into the attack, I needed to understand how certificates work. Certificates are at the core of **SSL/TLS**, which provides encryption and ensures secure communication between browsers and servers. At its core, a certificate consists of:

- **Public Key**: Used for encrypting data.
- **Private Key**: Kept secret and used for decrypting data.
- **Metadata**: Contains information about the certificate holder, including the website’s identity and a digital signature from a trusted Certificate Authority (CA).

In most cases, certificates are issued by trusted CAs like Let’s Encrypt or DigiCert. However, **GiftScheduler** used a **self-signed certificate** — meaning the website issued its own certificate instead of relying on a trusted third-party authority. This makes the website vulnerable because browsers do not automatically trust self-signed certificates. If someone were to intercept the traffic, they could potentially modify the data or steal sensitive information.

### **Setting Up the Attack**

To ensure that I wouldn’t leave any trace in Wareville’s DNS logs, I started by manipulating DNS traffic on my machine. I did this by adding the target domain (`gift-scheduler.thm`) to my **/etc/hosts** file and pointing it to the **AttackBox IP**.

Here’s the command I used:

```bash
root@attackbox:~# echo "MACHINE_IP gift-scheduler.thm" >> /etc/hosts
```

This allowed me to visit the **GiftScheduler** website without triggering any DNS logs, which is important for maintaining stealth in an attack. After making this change, I navigated to `https://gift-scheduler.thm` in my browser. As expected, I was greeted with a warning indicating that the website was using a **self-signed certificate**. This is where most users would just click through the warning without thinking twice, which is a critical mistake that I would use to my advantage.

### **Burp Suite Configuration: The Heart of the Attack**

With the website confirmed to be using a self-signed certificate, it was time to set up my attack environment. **Burp Suite** is a powerful tool that allows attackers to intercept, manipulate, and analyze web traffic. To carry out the MITM attack, I needed to route all traffic through my machine using Burp Suite as a proxy.

Here’s how I set it up:

1. **Start Burp Suite**: I ran Burp Suite by typing `burp` in the terminal. Once the tool launched, I accepted the default configuration and clicked "Next" to start the program.

2. **Configure the Proxy**: In Burp Suite, I selected the **Proxy** tab and turned off the **Intercept** option to prevent delays in the website responses. Then, I went to the **Proxy Settings** and added a new listener on **port 8080**. I set the listener to bind to the **AttackBox IP**, which allowed my machine to act as the middleman for all traffic.

### **MITM Attack in Action**

Now that Burp Suite was up and running, it was time to intercept traffic. To achieve this, I needed to reroute all traffic from Wareville through my machine. The key step here was to manipulate the **/etc/hosts** file again, but this time, I added a line to direct all requests for Wareville’s gateway to my **AttackBox**. Here’s the command I used:

```bash
root@attackbox:~# echo "CONNECTION_IP wareville-gw" >> /etc/hosts
```

By doing this, all requests from other machines in Wareville would now go through my AttackBox, and I would be in the middle of every interaction between the clients and the server. This allowed me to capture sensitive information, including **credentials** and **session tokens**.

### **Intercepting Sensitive Traffic**

With everything in place, I used a custom script to simulate traffic from other users interacting with the GiftScheduler site. The script continuously captured requests, and I could monitor the traffic using the **HTTP History** tab in Burp Suite. 

The real magic happened when I intercepted **POST requests** containing sensitive information like usernames and passwords. Since the website was using an insecure self-signed certificate, this data was being transmitted **in clear text**, making it extremely easy for me to extract sensitive credentials. As I kept an eye on the requests coming in, I was able to capture the login credentials of multiple users, including admin accounts.

This was a critical vulnerability because these clear-text credentials could now be used to gain unauthorized access to the website. The moment I obtained the admin credentials, I could disrupt the GiftScheduler’s operations, potentially ruining **G-Day** for everyone in Wareville.

### **Conclusion: The Evil Plan Unfolds**

By leveraging Burp Suite and exploiting the self-signed certificate vulnerability, I successfully intercepted sensitive data and gained access to the GiftScheduler website. With the admin credentials, I could’ve executed further malicious actions, like modifying the gift schedule or hijacking other accounts.

### **Key Takeaways**

1. **Self-Signed Certificates Are a Security Risk**: Self-signed certificates are not trusted by browsers by default, making them vulnerable to MITM attacks. Websites should always use certificates signed by a trusted Certificate Authority (CA).
   
2. **Burp Suite Is an Essential Tool**: Burp Suite is incredibly powerful for intercepting and manipulating traffic, especially in web application security testing. It allowed me to analyze requests, extract sensitive data, and perform the attack.

3. **MITM Attacks Are Effective**: By becoming the "middleman" between the client and the server, an attacker can easily intercept sensitive data like login credentials. It’s essential for websites to use **secure certificates** and **TLS encryption** to mitigate this risk.

4. **User Habits Are Dangerous**: In real-world scenarios, users often click through SSL warnings without verifying the authenticity of certificates. This is a dangerous practice that can be exploited by attackers.

### **Final Thoughts**

Well it reinforced the importance of proper certificate management and encryption in web security. As attackers, we can exploit weaknesses like self-signed certificates and **MITM attacks** to compromise sensitive data. As defenders, we must ensure that we use trusted CA certificates and employ **strong encryption** to protect against such vulnerabilities.

**Answers**

*What is the name of the CA that has signed the Gift Scheduler certificate?* **THM**

*Look inside the POST requests in the HTTP history. What is the password for the snowballelf account?* **c4rrotn0s3**
 
*Use the credentials for any of the elves to authenticate to the Gift Scheduler website. What is the flag shown on the elves’ scheduling page?* **THM{AoC-3lf0nth3Sh3lf}**
 
*What is the password for Marta May Ware’s account?* **H0llyJ0llySOCMAS!**
 
*Mayor Malware finally succeeded in his evil intent: with Marta May Ware’s username and password, he can finally access the administrative console for the Gift Scheduler. G-Day is cancelled! What is the flag shown on the admin page?* **THM{AoC-h0wt0ru1nG1ftD4y}**
