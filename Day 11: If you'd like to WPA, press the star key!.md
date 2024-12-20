## Tools Used

- **Aircrack-ng**: This suite of tools is used for monitoring wireless networks, capturing handshakes, and cracking WPA/WPA2 passwords.
- **iw**: A tool for interacting with wireless interfaces and setting them into monitor mode.
- **airodump-ng**: This tool is used to capture Wi-Fi network traffic, including handshakes.
- **aireplay-ng**: A tool used to inject deauthentication packets to force a client to reconnect to the network.

## Setup

Before diving into the attack itself, I first ensured that my environment was ready for wireless network analysis. I needed a wireless interface that could be set into **monitor mode** to capture network traffic. This mode allows a wireless interface to listen to all traffic on a specific channel, including the handshake I need to capture.

### Step 1: Set up the Wireless Adapter in Monitor Mode

To begin, I used the `iw dev` command to check the available wireless interfaces on my system. Then, I set my interface into monitor mode with the following commands:

```bash
sudo ip link set dev wlan2 down
sudo iw dev wlan2 set type monitor
sudo ip link set dev wlan2 up
```

This enabled my wireless adapter to listen to all network traffic on a specific channel, which was essential for the attack.

### Step 2: Scanning for Nearby Networks

Next, I needed to scan for available Wi-Fi networks. I used the following command to perform a scan and gather information about the available networks:

```bash
sudo iw dev wlan2 scan
```

This gave me a list of nearby networks, along with useful information like the SSID (network name), BSSID (MAC address of the AP), and encryption type. I identified the target network that I would be focusing on for the attack.

### Step 3: Capturing the 4-Way Handshake

Once I identified the target network, the next step was to capture the 4-way handshake between the client and the AP. The 4-way handshake is what allows the client and the AP to securely authenticate each other. To capture this handshake, I used `airodump-ng`:

```bash
sudo airodump-ng wlan2
```

This command showed all available networks and their associated clients. After identifying the target network, I ran the following command to focus on that network and start capturing the handshake:

```bash
sudo airodump-ng -c 6 --bssid 02:00:00:00:00:00 -w output-file wlan2
```

This command told `airodump-ng` to focus on channel 6 (the channel the target AP was on) and capture all traffic related to that network, saving the output to a file called `output-file`. If a client was connected to the network, it would try to reconnect, and during this process, the handshake would be captured.

### Step 4: Forcing a Reconnection Using Deauthentication Packets

If no clients were actively reconnecting, I needed to force one of the clients to disconnect and reconnect. This is where **aireplay-ng** came into play. I used the following command to send deauthentication packets to the connected client:

```bash
sudo aireplay-ng --deauth 10 -a 02:00:00:00:00:00 -c 02:00:00:00:01:00 wlan2
```

This command sent deauthentication packets to the target client, which disconnected the client from the AP. As a result, the client would try to reconnect, and during this process, the 4-way handshake would be captured by `airodump-ng`.

### Step 5: Cracking the WPA/WPA2 Password

Once I had the handshake captured, it was time to crack the password. I used a brute-force or dictionary attack to attempt to match the captured handshake against a list of potential passwords.

The cracking tool I used was **aircrack-ng**. The following command was used to start the cracking process:

```bash
aircrack-ng output-file.cap -w /path/to/wordlist.txt
```

Here, `output-file.cap` was the file containing the captured handshake, and `wordlist.txt` was a text file containing a list of common passwords. The tool would attempt each password in the list, checking if it would match the 4-way handshake.

If successful, the tool would reveal the WPA2 password. If not, I would need to try a larger or more specific wordlist until the correct password was found.

## Lessons Learned

Through this exercise, I learned a few valuable lessons about wireless network security:

1. **Weak Passwords Are Vulnerable**: The success of this attack largely depends on the strength of the password. A simple, common password is easy to crack with dictionary attacks. Strong passwords with a mix of characters, numbers, and symbols are essential to secure a WPA2 network.
   
2. **Use WPA3 If Possible**: WPA2, while still secure, is vulnerable to brute-force attacks like this one. WPA3, the latest Wi-Fi security standard, offers better protection and should be adopted wherever possible.

3. **The Importance of Monitor Mode**: Using monitor mode for network analysis is crucial in capturing unicast packets and handshakes. It’s a vital step in performing wireless attacks or auditing Wi-Fi security.

4. **Ethical Hacking and Legal Risks**: Always remember to only perform these types of attacks in environments where you have explicit permission to test the network. Unauthorized network access is illegal and unethical.

## Conclusion

SoI demonstrated the process of performing a WPA/WPA2 cracking attack, which involves capturing the 4-way handshake and attempting to crack the password using brute-force or dictionary attacks. This is a fundamental skill for network security professionals, as it allows them to test the strength of wireless network encryption and emphasize the importance of secure passwords.

**Answers**

*What is the BSSID of our wireless interface?* **02:00:00:00:02:00**
 
*What is the SSID and BSSID of the access point? Format: SSID, BSSID* **MalwareM_AP, 02:00:00:00:00:00**
 
*What is the BSSID of the wireless interface that is already connected to the access point?* **02:00:00:00:01:00**
 
*What is the PSK after performing the WPA cracking attack?* **fluffy/champ24**
