<img width="940" height="613" alt="image" src="https://github.com/user-attachments/assets/b421d75b-74f2-4472-9198-68fc6caa775d" /><img width="940" height="613" alt="image" src="https://github.com/user-attachments/assets/b421d75b-74f2-4472-9198-68fc6caa775d" /><img width="940" height="516" alt="image" src="https://github.com/user-attachments/assets/30b7724b-2a71-4c5d-927a-2885754828b7" /><img width="940" height="516" alt="image" src="https://github.com/user-attachments/assets/30b7724b-2a71-4c5d-927a-2885754828b7" /><img width="940" height="477" alt="image" src="https://github.com/user-attachments/assets/cc81c320-7e2c-4646-b2e9-ee8cc79ed06d" /><img width="940" height="477" alt="image" src="https://github.com/user-attachments/assets/cc81c320-7e2c-4646-b2e9-ee8cc79ed06d" /><img width="940" height="178" alt="image" src="https://github.com/user-attachments/assets/c6425799-9e4c-4ce4-9a42-9de68428a1e5" /><img width="940" height="178" alt="image" src="https://github.com/user-attachments/assets/c6425799-9e4c-4ce4-9a42-9de68428a1e5" /><img width="940" height="293" alt="image" src="https://github.com/user-attachments/assets/9b75eac0-e0cc-494e-b137-69aa35a9da45" /><img width="940" height="293" alt="image" src="https://github.com/user-attachments/assets/9b75eac0-e0cc-494e-b137-69aa35a9da45" /><img width="940" height="420" alt="image" src="https://github.com/user-attachments/assets/a237feb0-5a7d-48f2-9229-801ceaf5df41" /><img width="940" height="420" alt="image" src="https://github.com/user-attachments/assets/a237feb0-5a7d-48f2-9229-801ceaf5df41" />Hack The Box Writeup: Whisper
Challenge: Whisper
Platform: Hack the Box
Category: Digital Forensics & Incident Response (DFIR)

Scenario
On 21st January 2025, the SOC team detected suspicious activity originating from an employee workstation named Alpha. Since Alpha had no authorization to perform offensive or hacking-related activities, the incident immediately raised red flags.
The Incident Response team isolated the machine, and at that point, it became my responsibility to triage the system, analyse forensic artifacts, reconstruct the attacker’s actions, and determine the overall impact.

Tools I Used
During my investigation, I relied heavily on industry-standard DFIR tools, including:
•	Eric Zimmerman’s Tools (Registry Explorer, PECmd, MFTECmd, Timeline Explorer)
•	MFTExplorer
•	Shellbag Explorer
•	SQLite3
•	Impacket (secretsdump.py)
•	Hashcat
These tools allowed me to correlate registry, file system, browser, and security log artifacts into a clean timeline.

Question 1
What is the hostname of the affected machine?
To start the investigation, I wanted to firmly identify the compromised host. I loaded the SYSTEM registry hive (SYSTEM.DAT) into Registry Explorer.
By navigating to:
SYSTEM\ControlSet001\Control\ComputerName\ComputerName
I found the value that Windows uses to identify itself on the network. Seeing this felt reassuring because it confirmed I was analyzing the correct system.
<img width="940" height="601" alt="image" src="https://github.com/user-attachments/assets/16e3274a-913d-4ea6-a07b-b34f4964a0ab" />

Hostname: HELPDESK

Question 2
What IP address was assigned to the machine?
Next, I focused on the machine’s network identity. Still inside the SYSTEM hive, I navigated to:
SYSTEM\ControlSet001\Services\Tcpip\Parameters\Interfaces
Each GUID represented a network interface, so I carefully inspected them until I located the active one. The DhcpIPAddress field clearly showed the last IP lease issued to the machine.
<img width="940" height="420" alt="image" src="https://github.com/user-attachments/assets/a237feb0-5a7d-48f2-9229-801ceaf5df41" />
IP Address: 192.168.159.199
At this point, I knew exactly how this host appeared on the internal network.


Question 3
What is the Security Identifier (SID) of the machine?
Understanding SIDs is crucial because Windows tracks almost everything using them. I examined the registry and identified the SID associated with user activity on the system.
<img width="940" height="293" alt="image" src="https://github.com/user-attachments/assets/9b75eac0-e0cc-494e-b137-69aa35a9da45" />

The full user SID I found was:
S-1-5-21-953115734-2025219997-3674921352-1001
Since the RID (1001) represents the user account, I removed it to derive the machine SID:
S-1-5-21-953115734-2025219997-3674921352
This was a key moment because it allowed me to confidently correlate future events back to this system.

Question 4
What timezone was the machine configured with?
Timestamps mean nothing without knowing the timezone, so I immediately checked:
SYSTEM\CurrentControlSet\Control\TimeZoneInformation

<img width="940" height="303" alt="image" src="https://github.com/user-attachments/assets/c571c1f6-aa28-4b43-9357-99175e3cae18" />
The value clearly showed that the system was configured to Pacific Standard Time (PST).
This was important because it ensured my timeline reconstruction would be accurate.

Question 5
Which tool was used to dump credentials?
I explored the user’s Recent files directory:
C:\Users\A7md-NaS3eR\AppData\Roaming\Microsoft\Windows\Recent
When I spotted mimikatz.exe, it instantly stood out. Seeing this tool immediately confirmed malicious intent — there’s no legitimate reason for it to be there.
Credential dumping tool used: Mimikatz

Question 6
When was the tool last executed?
To verify execution timing, I turned to Windows Prefetch files, which are extremely reliable for execution evidence. I found two relevant files:
•	MIMIKATZ.EXE-7A038744.pf
•	MIMIKATZ.EXE-D870B873.pf
Using PECmd, I extracted the last execution timestamp.
<img width="631" height="395" alt="image" src="https://github.com/user-attachments/assets/147f8339-3d7a-4674-9a47-2ef8599e07ac" />

Last execution time:
2025-01-21 00:21:41
At this point, the attack timeline was becoming very clear.

Question 7
When was the tool first downloaded?
I wanted to know when the attacker initially brought Mimikatz onto the system. Using MFTExplorer, I located the file entry and reviewed its creation timestamp.
<img width="940" height="178" alt="image" src="https://github.com/user-attachments/assets/c6425799-9e4c-4ce4-9a42-9de68428a1e5" />

Download timestamp:
2025-01-21 00:20:15
The short gap between download and execution showed how quickly the attacker acted.

Question 8
From where was the domain-scanning script downloaded?
To answer this, I analyzed Microsoft Edge browser history, located at:
C:\Users\Alpha\AppData\Local\Microsoft\Edge\User Data\Default\History
Using SQLite3, I ran a query that converted Edge’s WebKit timestamps into readable time.
The result revealed the exact download source:
URL:
https://mega.nz/file/I31zBZTb#H5aUJamEj2xMroi9baoTKCMjQF1jiV8TK9gbjoNGgkw
This confirmed external data exfiltration activity.
Question 9
What is the full path of the downloaded script?
Checking the Downloads directory revealed the PowerShell script used for domain scanning.
File path:
C:\Users\Alpha\Downloads\DC-Scan.ps1

Question 10
When was the script deleted?
To recover deletion activity, I parsed the NTFS USN Journal ($J) using MFTECmd, exported the results, and analyzed them in Timeline Explorer.
After filtering for .ps1 files, I located the delete event.

<img width="940" height="477" alt="image" src="https://github.com/user-attachments/assets/cc81c320-7e2c-4646-b2e9-ee8cc79ed06d" />

Deletion timestamp:
2025-01-21 01:27:47
Even after deletion, the filesystem left behind undeniable evidence.


Question 11
What shared file did the attacker access?
Using Shellbag Explorer, I reconstructed folder and network share navigation. Combined with Recent Documents, one file clearly stood out.
<img width="940" height="571" alt="image" src="https://github.com/user-attachments/assets/af6c1639-d6f5-440d-812f-13f008fb2cf5" />
<img width="940" height="516" alt="image" src="https://github.com/user-attachments/assets/30b7724b-2a71-4c5d-927a-2885754828b7" />

Accessed file:
\\CYBERUP\Users\Administrator\Desktop\Top-Secret\Secret.xlsx
This was a critical finding because it demonstrated lateral access to sensitive data.

Question 12
What SID was assigned to the newly created user?
To identify persistence mechanisms, I reviewed Security Event Logs, filtering for Event ID 4720 (new account creation).
I found a single event confirming the creation of a new account named Admin.
<img width="940" height="613" alt="image" src="https://github.com/user-attachments/assets/b421d75b-74f2-4472-9198-68fc6caa775d" />

New account SID:
S-1-5-21-953115734-2025219997-3674921352-1002

Question 13
How many times did the new user log in?
I filtered Event ID 4624 (successful logon) for the Admin account SID.
No matching events appeared.
Total successful logins: 0
This told me the attacker created the account for persistence but never actually used it.

Question 14
What was the password of the new account?
This part felt especially rewarding. Using the SAM and SYSTEM hives, I ran Impacket’s secretsdump.py to extract local password hashes.
The hash for the Admin account was:
58a478135a93ac3bf058a5ea0e8fdb71
I cracked it using Hashcat with the rockyou.txt wordlist.
After cracking, the password appeared almost instantly — a classic weak choice.
Recovered password: Password123



                                                                             END












