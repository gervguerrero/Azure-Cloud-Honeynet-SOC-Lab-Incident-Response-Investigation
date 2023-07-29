# Azure-Cloud-SOC-Lab-Incident-Response

Builds off repository Azure-Cloud-SOC-Lab showcasing the Incident Response process Microsoft Sentinel  
![final map](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/c1ef655b-4ebf-4b86-b7d7-3060246645a6)

This page builds off of my [Azure-Cloud-SOC-Lab](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab/tree/main). 

In Microsoft Azure, I built a public facing Honey Net to attract real world cyber attackers to monitor their TTPs. Here I showcase basic methods to view the incidents caused by these threat actors in Microsoft Azure and implement controls to emulate the incident response process from NIST 800-61 used by Security Operation Center (SOC) Teams. 

This Incident Response process is a very basic demonstration of how a malicious cyber actor can brute force into an unprotected system that is left open, and how it is observed in Microsoft Sentinel and the Log Analytics Workspace in a Microsoft Azure Cloud environment. 

<br/> 


# Vulnerable Network 
![uncsecure net map](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/97549972-f13c-445c-9719-38f30ecf44ae)

The Azure cloud environment was left wide open without any real protection from cyber threat actors on the internet. The map shows  5 layers of defenses that were left open for threat actors to directly connect to the virtual machines and brute force/login into.

1. Azure Firewall
2. Network Service Group (NSG) covering the entire virtual network
3. Network Service Group specific to each virtual machine
4. Operating System specific firewalls to each virtual machine and firewall rules specific to the Blob Storage/Key Vault
5. Private Endpoint Protection for Blob Storage/Key Vault
   
<br/> 

# Start of the Incident Response Process 
![IR](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/fdbc811a-f5bd-469b-9091-5dff69e0bf47)

For this specific incident, NIST 800-61's Incident Response model was used. 

## Step 1: Preparation

A majority of this step was covered in our Honeynet and SOC building period. The important part of this step in this environment was ingesting logs from our assets and importing them into our Log Analytics Workspace to be used by Microsoft Sentinel.
Click here to see the process used for this project: (PLACEHOLDER)

The main goals in this step is to gather as much information as possible before jumping into the analysis step. The ingestion of logs in any incident response or network assessment is crucial to a successful analysis of a network. Endpoint logs give your security team the telemetry and visibility that often isn't seen if only network traffic is collected, or if the security team isn't able to deploy Endpoint Detection (EDR) tools. 

In a real life Incident Response, here are some other items to consider for the preparation phase:

- Communication Plans with Key Stakeholders, Network Owners, and Security Response Team
- Acquisition of Network Map and Network Scope, IP/Inventory/Asset List
- Scoping Document/Questionnaire covering Pattern of Life/Baseline for the Network (When/How are backups performed? How many Administrators does the network Have?)
- Authorized Software List, Users List
- Identification of Crticial Assets to the Network Owners
- Developing Security Sensor Strategy, what kind of Security Tools to use, how the data will be collected for analysis by the Security Team
- Testing SIEM, Log Collection, EDR tools prior to deployment
- Reviewing data retention plan and ensuring sufficient storage for data collection 
- Threat Intelligence published to the Information Security Community (APTs, widely known CVEs, information that might be relevant to the network if it is specific like Hospitals, Bank, Schools, etc.)
- All Data surrounding or related to the Initial Event found that led to an escalated Incident. 


## Step 2: Detection & Analysis 

The main goal of the Detection and Analysis phase is to validate the incident reported, determine the scope of impact, and to assign the severity level. This is heavily impacted by the amount of work done during the preperation phase, whether it's gathering accurate network architecture information from the network owners/adminsitrators, or making sure your network defender tools have been tested and are functional before deployment. You want ensure in the Detection & Analysis phase that you and your team are investigating the incident with accurate data from the start, and with tools that are operational. Time is critical when investigating a high priority incident, and the security team needs to progress forward in this phase without backtracking due to inaccurate information or malfunctioning tools. Though these barriers can happen in this phase or any phase, they should be taken care of as much as possible in the Preparation phase for a smoother overall investigation for the Detection & Analysis phase.

Below are some other items to consider in the Detection & Analysis Phase:  

- Verification of proper SPAN/Mirror Port collection, or TAP collection point (Sensor Strategy)
- Verification of Log ingestion into SIEM
- Verification of EDR/Software deployed properly onto endpoints
- Incident Identification and Prioritization
- Incident Triage
- Incident Validation/Investigation
- Proactive Threat Hunting with other Network Anomalies or Indicators
- Proper Incident Documenting/Reporting
- Communication and Collaboration Methods

### Investigating The Incident

The main incident that we will be focusing on is a Brute Force Login success to the Windows 10 Virtual Machine. It's important to note that while the logs and alerts generated in the environment are from real cyber threats brute forcing the public facing virtual machines, this attack was generated by myself failing to Remote Desktop into the Windows 10 VM around 10 times and subsequently suceeding a login with the correct credentials. The logic of the analytical rule capturing this action triggered an Incident alerting a successful brute force login seen in Microsoft Sentinel. 

Below is the Microsoft Sentinel Incidents page and the KQL (Kusto Query Language) query to trigger this action simulating a real brute force login. 

![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/5300957f-3357-4488-94ac-3c666a3344b6)


