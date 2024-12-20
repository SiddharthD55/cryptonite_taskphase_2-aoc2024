Through this thing, I explored how WebSocket connections can be exploited through message manipulation. WebSocket Message Manipulation involves intercepting and altering messages between a client and server over an open connection. The goal here is to demonstrate how such an attack works and how easily it can be performed if proper security measures aren't implemented.

## What Are WebSockets?

Before diving into the attack, let's break down what WebSockets are. WebSockets are a protocol that provides full-duplex communication channels over a single, long-lived connection. Unlike the traditional HTTP request-response model, WebSockets allow continuous communication between the client and server. Once a WebSocket connection is established, data can flow in both directions at any time without needing to re-establish a connection.

While WebSockets are incredibly efficient for real-time data, they also come with risks if not handled properly. Some of the vulnerabilities that make WebSockets prone to exploitation include:

- **Weak authentication and authorization**: WebSockets don’t have built-in mechanisms for handling user authentication or session validation, which means attackers can impersonate legitimate users if the app isn’t secured.
- **Message tampering**: If there’s no encryption, attackers can intercept and modify messages in transit, altering the data being exchanged.
- **Cross-Site WebSocket Hijacking (CSWSH)**: Attackers could trick the browser into opening a WebSocket connection to a malicious server and hijack it.
- **Denial of Service (DoS)**: Open WebSocket connections are susceptible to DoS attacks, where the attacker floods the server with excessive messages, potentially crashing it.

## Setting Up the Attack

### Tools Used

- **Burp Suite**: For intercepting and modifying WebSocket traffic. Burp Suite is an essential tool for any security tester working with web applications. It allows you to capture and manipulate requests and responses, including WebSocket messages.
- **AttackBox**: An isolated environment for running the attack. It ensures no unintended network traffic is exposed during the testing process.

### Steps to Exploit

1. **Start the Web Application**: 
   I started by launching the vulnerable web application, in this case, the "Reindeer Tracker" app, which allows users to track cars. The app uses WebSockets to maintain real-time communication.

2. **Configure Burp Suite**:
   In Burp Suite, I configured the proxy settings to intercept traffic between the browser and the server. This step is crucial because Burp Suite acts as a man-in-the-middle (MITM), allowing me to capture and alter the WebSocket messages in real time.

3. **Initiate WebSocket Communication**:
   With the application running, I clicked the “Track” button in the app, which triggers a WebSocket connection to start tracking a user's car. Burp Suite immediately intercepted the traffic.

4. **Message Manipulation**:
   The intercepted message contained a `userId` parameter. Initially, the `userId` was set to 5, representing the user I was tracking. I modified this value to 8 to simulate tracking a different user.

5. **Forward the Message**:
   After making the changes, I forwarded the modified WebSocket message through Burp Suite. The server processed this new message, and on the front end, I could see that the app was now tracking the user with ID 8 instead of 5.

6. **Verifying the Impact**:
   The app updated the user being tracked, confirming that the message manipulation was successful. I had effectively hijacked the WebSocket communication by altering the `userId` without any permission, which shows how easily an attacker could exploit this vulnerability in a real-world application.

## Why is This Dangerous?

WebSocket Message Manipulation can lead to several security issues:

- **Unauthorized Actions**: Attackers can impersonate users and carry out actions on their behalf, such as transferring funds or changing account settings.
- **Privilege Escalation**: If an attacker can modify the message, they might gain unauthorized access to admin privileges or restricted areas of the app.
- **Data Corruption**: Altering messages can corrupt critical data, affecting the integrity of the entire system.
- **Denial of Service**: Flooding the server with manipulated WebSocket messages can overload it, causing the app to crash.

This vulnerability is especially dangerous in applications dealing with sensitive data, financial transactions, or user accounts. Without proper security, WebSockets become a potential gateway for attackers to gain control over the app’s functionality.

## Mitigating the Risk

To prevent WebSocket message manipulation, developers must implement the following best practices:

- **Authentication and Authorization**: Ensure that WebSocket connections are properly authenticated and that users are authorized to perform the actions they are requesting.
- **Message Integrity**: Use message signing or hashing to ensure that messages cannot be tampered with while in transit.
- **Encryption**: Always use SSL/TLS to encrypt WebSocket communication, ensuring that the messages are protected from interception or alteration.
- **Input Validation**: Validate all incoming data on both the client and server to detect any anomalies or unauthorized requests.

## So...

WebSocket vulnerabilities, like message manipulation, are a serious threat if not properly addressed. In this writeup, I demonstrated how an attacker could intercept and modify WebSocket messages in real time to manipulate an app’s behavior. Understanding these vulnerabilities is key to securing applications that rely on WebSocket connections, especially for real-time or sensitive data exchanges.

By using tools like Burp Suite and following the steps outlined above, we can test and exploit WebSocket vulnerabilities. However, it’s important to always implement strong security measures to defend against these types of attacks in production environments.

**Answers**

*What is the value of Flag1?* **THM{dude_where_is_my_car}**

*What is the value of Flag2?* **THM{my_name_is_malware._mayor_malware}**
