Hack The Box Writeup: Whisper

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

SCREENSHOT LINK BELOW:-

https://drive.google.com/file/d/1_9rMGl7fjeIkWzmOs_zFQwl2hIHzLMZa/view?usp=drive_link

Hostname: HELPDESK    
