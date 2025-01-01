Well here I had to reverse engineer a suspicious binary file, `WarevilleApp.exe`. This was initiated due to alerts from McSkidy’s security dashboard, which detected anomalous activity on a file-sharing web application. This writeup documents my approach, findings, and reflections on the task, aiming to provide a comprehensive overview of the reverse engineering process.

## Tools and Setup

To tackle this task effectively, I relied on a carefully selected set of tools and a controlled environment, including:

- **PEStudio**: For static analysis of the Portable Executable (PE) file structure, offering a high-level overview of the file's properties.
- **ILSpy**: A powerful decompiler designed to analyze .NET binaries and provide insights into their high-level logic.
- **Sandbox Environment**: A secure setup for executing and observing the binary’s runtime behavior without risking system integrity.

The binary was provided in a controlled TryHackMe environment, ensuring safe handling of potentially malicious code. This setup minimized risks and maximized investigative efficiency.

## Step 1: Initial Analysis with PEStudio

The first step involved performing static analysis to gather foundational information about the `WarevilleApp.exe` file. Here’s what I did:

1. **File Loading**: Navigated to `C:\Users\Administrator\Desktop\` and opened the binary in PEStudio.
2. **General Properties Examination**: Extracted key metadata, including the SHA-256 hash, to ensure the file’s integrity and establish a baseline for further analysis. Identified that the binary was compiled using the .NET framework, making it suitable for decompilation.
3. **Section Analysis**: Focused on the `.text` section, containing the executable code, and noted its hash for later comparison. Additionally, the "Indicators" section flagged potential artifacts such as embedded URLs and function names.

This step provided a solid starting point, enabling me to identify areas for deeper inspection and verify the file’s structural integrity.

## Step 2: Decompiling with ILSpy

After completing the static analysis, I moved on to decompilation using ILSpy. This allowed me to explore the high-level logic of the binary. Upon navigating to `Program > Main`, I uncovered the following:

1. **Console Output Logic**: The application printed specific messages to the console, including an introductory greeting and updates on its progress.
2. **Key Variables**: The program initialized two significant variables:
   - `address`: A URL pointing to a PNG file.
   - `text`: The path on the Desktop where the PNG file would be saved after downloading.
3. **Core Functionality**:
   - Utilized the `WebClient` class to download the PNG file from the specified URL.
   - Opened the downloaded file using the default image viewer, likely Paint.
4. **Error Handling**: Included a try-catch block to gracefully manage exceptions during file download or execution.

The decompiled code revealed that the binary’s behavior was benign, designed to demonstrate file download and execution capabilities rather than execute malicious actions.

## Step 3: Dynamic Analysis

To validate the findings from static and decompiled analysis, I executed the binary in a sandbox environment:

1. **Execution Observations**: Upon running the file, it displayed the expected console messages, confirming the presence of user-facing output.
2. **File Download Verification**: Confirmed that the PNG file was downloaded to the Desktop as specified in the code.
3. **Runtime Behavior**: Observed the file opening in Paint, consistent with the default application for handling PNG files on the system.

The runtime behavior aligned perfectly with the logic extracted during decompilation, reaffirming that the binary posed no immediate threat.

## Broader Implications and Context

This exercise goes beyond just understanding a benign binary. It reflects the real-world importance of being prepared to tackle various types of executables, from harmless demonstrations to potentially harmful malware. In modern cybersecurity environments, professionals frequently encounter binaries that could be testing their vigilance or masking more complex threats. 

The structured methodology used here could serve as a blueprint for analyzing more sophisticated binaries in the future. Moreover, the identification of harmless binaries is equally valuable, as it prevents the allocation of resources to unnecessary investigations.

## Key Takeaways

This exercise highlighted several critical principles of reverse engineering:

- **Static Analysis Tools**: Applications like PEStudio are essential for gathering preliminary insights and identifying potential red flags in executable files.
- **Decompilation for Clarity**: Tools such as ILSpy enable the exploration of high-level code, offering valuable context for understanding a binary’s functionality.
- **Dynamic Verification**: Executing the binary in a controlled environment provides a final layer of assurance, confirming observed behaviors and dispelling doubts about potential threats.
- **Holistic Approach**: Combining static, decompiled, and dynamic analyses ensures a thorough and reliable understanding of executable files, equipping investigators to identify and mitigate potential risks effectively.

## Reflection and Learning

While the binary proved to be benign, this investigation underscored the importance of maintaining a critical and analytical mindset. Modern cybersecurity challenges demand a balance between technical proficiency and adaptive problem-solving. This exercise also reinforced the necessity of leveraging diverse tools to achieve comprehensive results. For example, while PEStudio laid the groundwork, ILSpy and dynamic analysis added depth and confidence to the investigation.

Engaging in this type of reverse engineering is not merely about solving isolated problems; it’s about building a skill set that evolves with emerging threats.

## In the end...

Although `WarevilleApp.exe` proved to be a harmless demonstration program, the process of reverse engineering it reinforced essential skills and best practices for analyzing binaries. This emphasized the importance of a systematic approach, leveraging diverse tools and methods to build a comprehensive understanding of executable files.

**Answers**

*What is the function name that downloads and executes files in the WarevilleApp.exe?* **DownloadAndExecuteFile**

*Once you execute the WarevilleApp.exe, it downloads another binary to the Downloads folder. What is the name of the binary?* **explorer.exe**
 
*What domain name is the one from where the file is downloaded after running WarevilleApp.exe?* **mayorc2.thm**
 
*The stage 2 binary is executed automatically and creates a zip file comprising the victim's computer data; what is the name of the zip file?* **CollectedFiles.zip**

*What is the name of the C2 server where the stage 2 binary tries to upload files?* **anonymousc2.thm**
