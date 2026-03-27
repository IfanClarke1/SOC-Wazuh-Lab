# SOC-Wazuh-Lab
This project demonstrates a simulated SOC (Security Operations Centre) environment using Wazuh as the SIEM (Security Operations and Event Management) tool.
I ran 3 virtual machines - an instance of Kali, Windows and Ubuntu with the open source SIEM Wazuh installed.

**Setup**
* 3 VMs (virtual machines) running on VMWare Workstation Pro
* Attack Machine: Kali Linux
* Target Machine: Windows 11
* SIEM Machine: Ubuntu running Wazuh

  **Attack Simulation**

The objective was to replicate a common attack scenario including reconaissance, carried out by Nmap scans, and brute-force authentication attempts using Hydra.

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

*Phase 3 - Initial Access*

I was not able to find a list of users using enumeration but, as the focus of this lab is defence, I decided to go ahead with using the correct username for a brute force attack with Hydra









