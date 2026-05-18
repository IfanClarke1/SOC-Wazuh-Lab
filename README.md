# SOC-Wazuh-Lab
This project demonstrates a simulated SOC (Security Operations Centre) environment using Wazuh as the SIEM (Security Incidents and Event Management) tool.
I ran 3 virtual machines - an instance of Kali, Windows and Ubuntu with the open source SIEM Wazuh installed.

**Setup**
* 3 VMs (virtual machines) running on VMWare Workstation Pro
* Attack Machine: Kali Linux
* Target Machine: Windows 11
* SIEM Machine: Ubuntu running Wazuh

  **Attack Simulation**

The objective was to replicate a common attack scenario including reconaissance, carried out by Nmap scans, and brute-force authentication attempts.

*Phase 1 - Reconnaissance* 

I ran the following Nmap scan to find open ports:

nmap -sS -T4 (target-ip)


<img width="290" height="114" alt="NMap Scan Results" src="https://github.com/user-attachments/assets/4fc998fe-2e47-4924-af23-a2aa746dabcb" />

From this I learned that the following ports were open: 135, 139, 445.

I identified port 445 (SMB) as a good attack vector

*Phase 2 - Enumeration*

Port 445 was identified as the attack surface of choice. SMB (Server Message Block) Enumeration allows me to query for details such as users and groups.

I tried both Nmap and Enum4linux to see if I could find a list of user accounts and shared resources.

These returned minimal information, indicating the target machine had restrictions in place to protect it against similar SMB queries

<img width="720" height="415" alt="Nmap Enumeration Results" src="https://github.com/user-attachments/assets/959a2928-96b6-4d02-a1a9-43b21a6ee96f" />

<img width="719" height="418" alt="Enum4Linux Scan Results" src="https://github.com/user-attachments/assets/ea391d76-5fa0-495c-84e6-0d987927af94" />



*Phase 3 - Initial Access*

From here I decided to manually replicate a brute-force attack by trying a series of incorrect password attempts.

 **Defense Simulation**

 The objective was to find evidence of the brute force in Wazuh.

 First step was to create a rule for an alert in Wazuh.

 I added the following to the Wazuh rules:

<img width="424" height="74" alt="image" src="https://github.com/user-attachments/assets/a50da056-ce30-48ed-9949-e615469aea77" />


This meant that an alert would be created if 5 incorrect password attempts were made within 60 seconds

**Key Takeaways**

The first thing that struck me is how enumaration returned little to no information. From doing a little research, I discovered that Windows 11 restricts anonymous SMB access by default, which is a sensible measure. This means an attacker would need valid credentials before running the brute force, making the attack more difficult and costly.

There is a lot of inbuilt good security measures in Windows 11 so the enumaration and brute force attempts I tried didn't work. Because of this, I had to go simple and try a load of wrong guessed passwords manually to recreate a brute force attack













