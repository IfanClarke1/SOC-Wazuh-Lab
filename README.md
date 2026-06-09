# SOC-Wazuh-Lab
This project demonstrates a simulated SOC (Security Operations Centre) environment using Wazuh as the SIEM (Security Incidents and Event Management) tool.
I ran 3 virtual machines - an instance of Kali, Windows and Ubuntu with the open source SIEM Wazuh installed.

**Setup**
* 3 VMs (virtual machines) running on VMWare Workstation Pro
* Attack Machine: Kali Linux
* Target Machine: Windows 11
* SIEM Machine: Ubuntu running Wazuh

**Network Diagram**

<img width="375" height="250" alt="Network Diagram Image" src="https://github.com/user-attachments/assets/5fd5f5e6-4bc8-4bc1-9b96-12bb0b81bc89" />


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

<img width="360" height="208" alt="Nmap Enumeration Results" src="https://github.com/user-attachments/assets/959a2928-96b6-4d02-a1a9-43b21a6ee96f" />

<img width="360" height="209" alt="Enum4Linux Scan Results" src="https://github.com/user-attachments/assets/ea391d76-5fa0-495c-84e6-0d987927af94" />

*Phase 3 - Initial Access*

From here I decided to manually replicate a brute-force attack by trying a series of incorrect password attempts in a span of 60 seconds. At first I attempted the brute force via Hydra trying a series of passwords on both RDP and SMB. I faced practical limitations due to the lack of information gathered from enumaration and so I pivotted to manually typing in incorrect passwords in Windows.


 **Defense Simulation**

 The objective was to find evidence of the brute force in Wazuh.

 First step was to create a rule for an alert in Wazuh.

 I added the following to the Wazuh rules:

<img width="424" height="74" alt="image" src="https://github.com/user-attachments/assets/a50da056-ce30-48ed-9949-e615469aea77" />

This meant that an alert would be created if 5 incorrect password attempts were made within 60 seconds

After the brute force was simulated the following alert appeared in Wazuh, as expected

<img width="677" height="103" alt="image" src="https://github.com/user-attachments/assets/b128729d-0755-49b6-925e-304600ec6d51" />

This means Wazuh successfully detected the brute force activity via integration with the Windows Security Event log. 5 instances of rule '60122' (a logon failure in Wazuh) running in the space of 60 seconds set off my custom rule. 

**Investigation Walkthrough**

*Overview*

During a controlled attack simulation, I identified and investigated a brute force authentication attack against a Windows 11 host. Wazuh created an alert based on a rule I created that established 5 incorrect passwords in 60 seconds was likely to be suspicious in nature. The following documents my investigation process as it would appear in a real SOC triage workflow.

*Initial Alert Triage*

The investigation began when Wazuh generated an alert on the dashboard. The alert was triggered by a custom rule I had created in `local_rules.xml`, set to fire when five or more failed authentication attempts were recorded against a single host within a 60-second window.

The alert presented with the following characteristics:

- Rule ID: 100100 (local_rules.xml)
- Alert level: 13 (high)
- Rule description: 5 failed Windows logons in 60s - possible brute force
- Destination IP: 192.168.x.x (Windows 11 — target VM)
- Timeframe: Repeated failures across approximately 45 seconds
- MITRE ATT&CK technique: T1110.001 — Brute Force: Password Guessing

The volume and speed of failures distinguished this as a user mistyping their password. A genuine failed login is usually only one or two events with a little more time in-between. 

*Event Log Analysis*

I then went from Wazuh to the Windows logs that Wazuh was reacting to. I found the following event ID 5 times in quick succession:

Event ID 4625 — Failed Logon

This was the primary signal. The logs showed a high volume of 4625 events in rapid succession, all originating from the same source IP and all targeting the same user account.

*Custom Rule Assessment*

The detection was made possible by a custom rule I created in `local_rules.xml` that I wrote to go alongside Wazuh's default behaviour. Wazuh's out-of-the-box rules will flag individual failed logons, but do not correlate them into a brute force pattern with configurable thresholds.

My rule monitored for repeated Event ID 4625 events, set a threshold of 5 failures, and applied a 60-second time window. When that threshold was crossed, Wazuh generated the alert that opened the investigation.

This approach reflects how detection engineering works in practice — default rules provide a baseline, but custom correlation logic is what catches attacks cleanly and with minimal false positives.

One consideration I noted: the 60-second window is well-suited for detecting fast, automated brute force (as Hydra produces), but a lower-velocity attack would evade this rule. A production environment would layer this with additional rules and policies. 

*SOC Handover Notes*
At 18:00, Wazuh generated a high-severity alert for repeated failed authentication against the Windows 11 host. Investigation of the underlying Windows Event logs confirmed a brute force attack. A total of 5 failed logon attempts (Event ID 4625) were recorded over approximately 45 seconds, consistent with automated tooling.

Recommended Actions: Implement or edit accoun lockout policy, reset credentials for the targeted account.


**Key Takeaways**

The first thing that struck me is how enumaration returned little to no information. From doing a little research, I discovered that Windows 11 restricts anonymous SMB access by default, which is a sensible measure. This means an attacker would need valid credentials before running the brute force, making the attack more difficult and costly.

There is a lot of inbuilt good security measures in Windows 11 so the enumaration and brute force attempts I tried didn't work. Because of this, I had to go simple and try a load of wrong guessed passwords manually to recreate a brute force attack













