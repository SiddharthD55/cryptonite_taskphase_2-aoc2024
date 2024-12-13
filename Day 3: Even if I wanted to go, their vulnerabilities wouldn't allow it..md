### **Investigating the Cyberattack on Frosty Pines Resort with ELK**

The Frosty Pines Resort website fell victim to a cyberattack exploiting an insecure file upload vulnerability, leading to Remote Code Execution (RCE). Using the ELK stack—Elasticsearch, Logstash, and Kibana—I analyzed the attack by investigating server logs. This writeup breaks down the key investigation steps under "Operation Blue" (log analysis) and "Operation Red" (vulnerability exploitation) to help understand how the attack unfolded and what was learned.


### **Operation Blue: Log Investigation with ELK**

#### **Step 1: Understanding ELK**  
The ELK stack is a powerful log management and analysis tool, composed of:  
- **Elasticsearch**: Indexes and stores logs, making them searchable.  
- **Logstash**: Processes and ships logs to Elasticsearch.  
- **Kibana**: Provides a visualization and querying interface for log analysis.

The investigation leveraged **Kibana's Discover feature** to explore logs and pinpoint key events. **Kibana Query Language (KQL)** was crucial in narrowing down logs to isolate suspicious activities.

#### **Step 2: Setting the Scene in Kibana**  
To analyze the attack, I accessed Kibana at `http://MACHINE_IP:5601/`, adjusted the time range to focus on October 1, 2024 (the attack date), and selected the "wareville-rails" log collection. This ensured that only relevant logs were examined.  

#### **Step 3: Filtering Logs with KQL**  
To locate malicious activities, I used KQL queries:  
- **Finding the Attacker's IP**:  
  `ip.address: "192.168.1.10"` filtered logs by the attacker's IP address.  
- **Tracking the Malicious File**:  
  `message: "shell.php"` identified logs referencing the uploaded web shell.  
- **Detecting Command Execution**:  
  `message: "exec"` revealed logs where the attacker executed commands.  

These queries provided insights into the attacker’s actions and timelines, including the file upload and subsequent exploitation.

#### **Step 4: Discovering the Malicious File Upload**  
The attacker uploaded a malicious file (`shell.php`) to the `/uploads` directory, leveraging an insecure file upload feature. Critical findings:  
- **File Name**: `shell.php`  
- **Timestamp**: October 1, 2024, at 14:32:10.  
- **Originating IP**: Logs confirmed the attacker’s IP as `192.168.1.10`.  

This confirmed that the attack began with a successful file upload bypassing validation controls.


### **Operation Red: Exploiting the Vulnerability**

#### **Step 1: Uploading the Malicious Web Shell**  
The attacker exploited an insecure file upload functionality, allowing them to upload a PHP file (`shell.php`) without restriction. This vulnerability arose from:  
- Lack of validation on file types.  
- Allowing files to be uploaded to an executable directory (`/uploads`).  

Once uploaded, the attacker accessed the shell via `http://resort-website/uploads/shell.php`.

#### **Step 2: Achieving Remote Code Execution (RCE)**  
The malicious `shell.php` enabled the attacker to execute commands on the server. Logs showed:  
- **Initial Commands**: The attacker ran commands like `whoami`, `id`, and `ls` to understand the system.  
- **Directory Traversal**: Commands such as `cd /var/www` and `ls -la` indicated the attacker exploring the file system.  

This exploitation allowed the attacker to gain complete control of the server and potentially exfiltrate sensitive data.


### **Exploitation Process and Findings**  

The attack followed a structured process:  
1. **Identifying the Vulnerability**: The attacker targeted the insecure file upload feature.  
2. **Uploading Malicious Script**: The file `shell.php` was uploaded to the `/uploads` directory.  
3. **Accessing the Web Shell**: The attacker navigated to the shell’s URL, enabling remote command execution.  
4. **Exploiting the Server**: Using RCE, the attacker explored the server, potentially compromising sensitive data or planting backdoors.  


### **Lessons Learned**

1. **Secure File Uploads**:  
   File upload functionality must validate input rigorously. Recommendations:  
   - Restrict acceptable file types and use MIME type validation.  
   - Rename uploaded files and remove executable permissions.  

2. **Monitoring and Logs**:  
   Regular log analysis using tools like ELK is essential for detecting malicious activities early. Kibana's Discover feature and KQL proved invaluable during the investigation.  

3. **Restrict Executable Directories**:  
   Uploaded files should be stored in non-executable directories to prevent malicious scripts from running.  

4. **Vulnerability Scanning**:  
   Regular vulnerability scans could have detected the insecure file upload feature, allowing the issue to be addressed before an attacker exploited it.  

5. **Incident Response Preparedness**:  
   Understanding how to use tools like ELK for incident response is critical for handling attacks effectively and minimizing damage.

### **Summary**

By investigating with ELK (Operation Blue), I identified key actions of the attacker, such as uploading the `shell.php` file and executing commands. Through Operation Red, I explored the exploitation process, from uploading the malicious file to achieving RCE. This investigation highlights the importance of secure development practices, proactive monitoring, and robust incident response plans to mitigate and prevent such attacks.

**Answers**
*BLUE: Where was the web shell uploaded to? Answer format: /directory/directory/directory/filename.php* **/media/images/rooms/shell.php**

*BLUE: What IP address accessed the web shell?* **10.11.83.34**

*RED: What is the contents of the flag.txt?* **THM{Gl1tch_Was_H3r3}**
