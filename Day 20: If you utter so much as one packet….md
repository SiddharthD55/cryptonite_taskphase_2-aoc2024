So here, what I learnt is...
- **Investigate network traffic using Wireshark**: I will analyze network packets and identify key patterns in the data stream.
- **Identify indicators of compromise (IOCs) in captured network traffic**: I will pinpoint signals that suggest a compromised system.
- **Understand how C2 servers operate and communicate with compromised systems**: I will explore how a Command and Control (C2) server communicates with infected systems.

## Connecting to the Machine
To begin my investigation, I first need to start the virtual machine. I can press the **Start Machine** button below to begin.

The machine will start in split-screen view. If I don’t see the VM, I can click the **Show Split View** button at the top of the page. Alternatively, I can connect to the VM via RDP using the credentials below:

- **Username**: Administrator
- **Password**: Commandncontrol001
- **IP Address**: MACHINE_IP

## Investigating the Depths

### Wireshark and C2 Communication
Wireshark is a tool I will use to analyze network traffic saved in a PCAP file. It is ideal for investigating how a compromised machine communicates with a Command and Control (C2) server. To understand this communication, I first need to know about the process:

- **C2 Communication**: After a system is compromised, the payload (malicious code) sent by the C2 server connects back to the server to receive instructions.
- **Beacons**: These are regular packets sent from the compromised system to the C2 server to confirm that the agent is still active and ready for commands. 

Using Wireshark, I’ll be able to inspect this traffic, track the beaconing process, and discover potential malicious activity.

## Diving Into the PCAP File

I will now dive deeper into the investigation by opening the PCAP file “C2_Traffic_Analysis” located on the Desktop. Double-clicking this file opens it in Wireshark.

Next, to focus on the traffic from the compromised machine, I will filter the packets by its IP address. I’ll do this by entering `ip.src == 10.10.229.217` in the Display Filter Bar and pressing **Enter**. This allows me to focus only on outbound traffic, which is where I expect to find communication with the C2 server.

## Analyzing the Captured Packets

### Highlighted Packets
As I scroll through the filtered traffic, I see some packets that are highlighted with arrows, signaling that they could be significant. The packet labeled **POST /initial** caught my attention. This packet likely contains the initial connection from the compromised machine to the C2 server.

#### Step 1: Inspect the Packet
By selecting the **POST /initial** packet (Frame 440), I can view detailed information at the bottom panes of Wireshark. Here, I will focus on the **Packet Bytes** pane, which shows the bytes used in the communication in both hexadecimal and ASCII formats. In the ASCII section, I notice the text **“I am in Mayor!”**. This is an important clue—it indicates that the payload has successfully connected to the C2 server.

### Following the HTTP Stream
To get the full context of the communication, I will right-click on the **POST /initial** packet and choose **Follow > HTTP Stream**. This displays the full HTTP conversation between the compromised machine and the C2 server.

In the **HTTP Stream**, I see that after sending the message **“I am in Mayor!”**, the C2 server responds with **“Perfect!”**. This confirms that the C2 server has received the initial connection and is ready to issue further commands.


### Analyzing Additional Packets
I’ll continue my investigation by reviewing other packets sent to the same destination IP. One such packet is the **GET /command** (Frame 457). After following the HTTP Stream, I discover that the server sends a **command** to gather information about the compromised machine, specifically by requesting the **current user’s information**. This type of request is typical in the reconnaissance phase, where attackers gather details about the system before executing more malicious commands.

### Investigating Exfiltration
The next significant packet is the **POST /exfiltrate** packet (Frame 476). Following the HTTP Stream here reveals that a file has been exfiltrated from the compromised system to the C2 server. This packet confirms that data is being stolen from the system, which is one of the main goals of many cyberattacks.

## Understanding the Beacon

Beacons are regular status updates sent by the compromised machine to the C2 server, often at regular intervals. These packets serve as a "heartbeat" to ensure the C2 server knows the agent is still active and listening for further commands.

The beacon typically contains encrypted information. In this case, I suspect that the beacon is using an encryption algorithm to keep its contents secret. Fortunately, I have the key to decrypt the beacon, which will allow me to reveal the contents.

## Decrypting the Beacon Using CyberChef

To decrypt the beacon, I will use **CyberChef**, a tool that can handle a variety of encoding, encryption, and decryption tasks. Here’s how I’ll decrypt the beacon:

1. **Open CyberChef** in my browser (the link is provided, but since the VM has no internet connection, I’ll need to open it outside the VM).
2. In the **Operations** pane, I will search for **AES Decrypt** and drag it to the **Recipe** area.
3. In the **Recipe** pane, I’ll set the mode to **ECB** and enter the provided decryption key.
4. In the **Input** pane, I will paste the encrypted beacon string.
5. I will then click **Bake** to decrypt the message.

The **Output** pane will display the decrypted contents of the beacon, revealing the communication between the compromised system and the C2 server.

## Conclusion
Through this, I’ve uncovered key insights into how a compromised machine communicates with its C2 server. By using Wireshark to analyze network traffic and CyberChef to decrypt beacons, I’ve been able to identify important clues, such as initial connections, commands, and exfiltrated data. This information is critical for understanding the nature of the attack and how it spreads through a network.

**Answers**

*What was the first message the payload sent to Mayor Malware’s C2?* **I am in Mayor!**
 
*What was the IP address of the C2 server?* **10.10.123.224**
 
*What was the command sent by the C2 server to the target machine?* **whoami**
 
*What was the filename of the critical file exfiltrated by the C2 server?* **credentials.txt**
 
*What secret message was sent back to the C2 in an encrypted format through beacons?* **THM_Secret_101**
