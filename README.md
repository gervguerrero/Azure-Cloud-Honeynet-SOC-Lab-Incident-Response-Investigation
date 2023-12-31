# 🔍Azure-Cloud-SOC-Lab-Incident-Response-Investigation🔍

This page showcases the Incident Response process using Microsoft Sentinel building off my repository [Azure-Cloud-Honeynet-SOC-Lab](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab/tree/main).

![final map](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/c1ef655b-4ebf-4b86-b7d7-3060246645a6)


In Microsoft Azure, I built a public facing Honeynet to attract real world cyber attackers to monitor their TTPs. Here I showcase basic methods to view the incidents caused by these threat actors in Microsoft Azure and implement controls to emulate the incident response process from NIST 800-61 used by Security Operation Center (SOC) Teams. 

This Incident Response process is a very basic demonstration of how a malicious cyber actor can brute force into an unprotected system that is left open, and how it is observed in Microsoft Sentinel and the Log Analytics Workspace in a Microsoft Azure Cloud environment. 

<br/> 


# Vulnerable Network 
![uncsecure net map](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/97549972-f13c-445c-9719-38f30ecf44ae)

The Azure cloud environment was left wide open without any real protection from cyber threat actors on the internet. The map shows  5 layers of defenses that were left open for threat actors to directly connect to the virtual machines and brute force/login into.

1. Azure Firewall
2. Network Service Group (NSG) covering the entire virtual network
3. Network Service Group specific to each virtual machine
4. Operating System specific firewalls to each virtual machine and firewall rules specific to the Blob Storage/Key Vault
5. Private Endpoint Protection for Blob Storage/Key Vault (Not configured, not shown on map.) 
   
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

The main goal of the Detection and Analysis phase is to validate the incident reported, determine the scope of impact, and to assign the severity level. This is heavily impacted by the amount of work done during the preparation phase. 

You want to ensure in the Detection & Analysis phase that the team is investigating the incident with accurate data from the start and with functioning tools. Time is critical when investigating a high priority incident, and the security team needs to progress forward in this phase without backtracking due to inaccurate information or malfunctioning tools. Though these barriers can happen in any phase, proper work in the Preparation phase leads for a smoother investigation for the Detection & Analysis phase.

Below are some other items to consider in the Detection & Analysis Phase:  

- Verification of proper SPAN/Mirror Port collection, or TAP collection point (Sensor Strategy)
- Verification of Log ingestion into SIEM
- Verification of EDR/Software deployed properly onto endpoints
- Incident Identification and Prioritization
- Incident Triage
- Incident Validation/Investigation
- Endpoint Forensics/Malware Analysis for Escalated Incidents 
- Proactive Threat Hunting with other Network Anomalies or Indicators
- Proper Incident Documenting/Reporting
- Communication and Collaboration Methods

### Investigating The Incident

The main incident that we will be focusing on is a Brute Force Login success to the Windows 10 Virtual Machine. It's important to note that while the logs and alerts generated in the environment are from real cyber threats brute forcing the public facing virtual machines, this specific attack was generated by myself failing to Remote Desktop into the Windows 10 VM around 10 times and subsequently succeeding a login with the correct credentials. The logic of the analytical rule capturing this action triggered an Incident alerting a successful brute force login seen in Microsoft Sentinel. 

Below is the Microsoft Sentinel Incidents page and the KQL (Kusto Query Language) query to trigger this action simulating a real brute force login. 

![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/5300957f-3357-4488-94ac-3c666a3344b6)

This is the KQL query that triggered the alert:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/10b72a06-3ff0-41c6-a7be-742b1acf5402)

```
// Brute Force Success Windows
let FailedLogons = SecurityEvent
| where EventID == 4625 and LogonType == 3
| where TimeGenerated > ago(1h)
| summarize FailureCount = count() by AttackerIP = IpAddress, EventID, Activity, LogonType, DestinationHostName = Computer
| where FailureCount >= 5;
let SuccessfulLogons = SecurityEvent
| where EventID == 4624 and LogonType == 3
| where TimeGenerated > ago(1h)
| summarize SuccessfulCount = count() by AttackerIP = IpAddress, LogonType, DestinationHostName = Computer, AuthenticationSuccessTime = TimeGenerated;
SuccessfulLogons
| join kind = inner FailedLogons on DestinationHostName, AttackerIP, LogonType
| project AuthenticationSuccessTime, AttackerIP, DestinationHostName, FailureCount, SuccessfulCount
```
<br/>
<br/>

Upon viewing the incident, we can set the incident an owner, a status, and a severity level in Microsoft Sentinel:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/7fe84c65-3424-4583-935d-8084861c3daf)

<br/>
<br/>

This is what the incident looks like when clicking on a single incident out of the list:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/0e5acec3-a986-40da-9e5e-830dbfa2715e)

<br/>
<br/>

Here we observe the Activity Log for the history of incident as it's administratively maintained:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/e6164b21-cf60-451b-bacb-cca3c33492af)

<br/>
<br/>

We can also see an Overview of this incident, the entities involved, and other incidents that Microsoft Sentinel relates as similar:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/f4139b3d-54e8-42a8-aa51-7aead412759f)

<br/>
<br/>

If the entities tab is expanded, we can explore details about the devices involved in this incident, both external and internal to the azure network affected:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/f1a5ef4b-0d0d-4d39-bf17-2c12439f6aa9)
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/92bb6f4e-3504-4f32-add7-e4527e15040f)

<br/>
<br/>

In the main incident page for the brute force, if we can click on the blue investigate button, we can see a high level overview map of the incident to determine the scope of impact:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/9487238a-84f1-426c-88b7-824d83371d9e)
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/b676afa5-b316-49b1-9095-87b166e1c135)

<br/>
<br/>

In this map, we can inspect the entities and see if there are any related events to an entity:

![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/ad71ae35-d09c-4795-aeea-833b0b9040bf)

![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/b5d0fc8b-a17a-4453-bf45-24d5ab81cf0f)

<br/>
<br/>

We can see a brute force attempt that this external IP performed as well and how it relates to the overall incident. (This was the attempt prior to the successful login.) 
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/7b3a5149-a6ce-4551-b0c7-06eb6fb3a036)

<br/>
<br/>

The other related events:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/0e413c44-50c5-4f87-86db-db1292d73a05)
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/e84ef84a-922f-456c-a71a-c29ce2367177)

<br/>
<br/>

The overall map with other events from the external IP expanded:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/56fa1a1f-b6dc-42f2-830d-6ec8ada16d89)
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/05676c2f-eae6-42f6-976c-9f25ad577542)

<br/>
<br/>

If we expand the related events from the victim entity the Windows 10 virtual machine, we can see this machine is involved in several incidents:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/cec2f243-00ea-4f52-904f-36995079cd15)
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/8b42cb07-be3a-4afb-9a33-7660d6244e44)

<br/>
<br/>

After getting a good understanding in the incident page, we need to determine the legitimacy of the incident and label it as either a True Positive, False Positive. To do this we view the Logs that generated this alert in the Log Analytics workspace by using the KQL query listed earlier above:
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/23e6cfcd-419b-4b17-91d4-981fbf100514)

<br/>
<br/>

Expanding these logs returned by this query reveal the IP seen in the Incident page and map and it's details on it's brute successful brute force:

![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/6a7b4e4c-3e46-4e3f-a066-7fecbe439a5b)
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/147f4d87-11c8-4540-93cf-c06d608cebbe)

<br/>
<br/>

We can get a little bit specific to validate the incident and see who else logged in recently as successful to this virtual machine. (Note that this KQL query is very simple and we are collecting Windows Event logs from only 1 VM in this project.)
![image](https://github.com/gervguerrero/Azure-Cloud-SOC-Lab-Incident-Response/assets/140366635/5ee62c39-cb39-4642-8097-abbc9607b60b)

At this point we can confirm that the Brute Force Success was a True Positive. The security team would then perform proactive threat hunting to any other machines related to the victim or servers that it commonly authenticates with. With validating the incident, we move into the next phase of the incident response process.

## Step 3: Containment, Eradication, and Recovery
### Containment
From here, depending on how the security team operates, the computer being investigated would be isolated to contain any immediate threats, and a thorough digital forensics investigation would take place to fully understand the complete impact of this attack. The forensics investigation is a very detailed process that would require its own page to explain and is not covered in this exercise. 

Though we can quickly list out key areas of forensic investigation for responding to an incident in a Windows operating system:


**System & User Information**
- Registry
- File Analysis
- NTFS

**Evidence of Malware Execution**
- Background Activity Moderator (Information about executables)
- ShimCache (Information about executables and backwards compatibility) 
- AmCache (Information related to windows apps experience and compatibility) 
- Prefetch (Information about applications that executed) 


**Persistence Mechanisms**
- Run Keys
- Startup Folder
- Scheduled Tasks
- Services

Upon a full investigation and understanding what needs to be fixed, the eradication and recovery steps begin.

In this simple exercise, the only malicious event was a successful brute force login from an unauthorized external IP. No persistence mechanisms or elaborate vulnerabilities were exploited.

Now that we know what needs to be corrected, we can start fixing it.

### Eradication and Recovery

To harden our network against ONLY an unauthorized brute force login, we need to:
- Change the password to a more complex one meeting security requirements.
- Implement network controls to disable access to unauthorized parties. (Implement Firewall Controls)
- Capture logging to monitor and observe if security controls are successful.

Focusing on NIST's 800-53 R5 SC-7 Boundary Protection, the security controls implemented were 5 layers of defense in the Azure network:
1. Azure Firewall
2. Network Service Group (NSG) covering the entire virtual network
3. Network Service Group specific to each virtual machine
4. Operating System specific firewalls to each virtual machine and firewall rules specific to the Blob Storage/Key Vault
5. Private Endpoint Protection for Blob Storage/Key Vault 

**For the sake of keeping this page at an acceptable size, click here to observe the detailed hardening process on the Azure network with firewall protection**: 

[Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening)

Here are the before and after hardening Azure network maps for a visual aid:
![Unsecured Network](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Incident-Response-Investigation/assets/140366635/7a946412-7d5f-4f04-9dbe-416a6daff5db)
![Secured Network Azure firewall](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Incident-Response-Investigation/assets/140366635/2abaae70-59f4-4b9f-8abb-2f885b8ba99f)

## Step 4: Post-Incident Activity

After implementing and observing successful security controls, the post incident activity focuses on reviewing the incident, how it was corrected, and everything in between to make it better next time. Here are some key areas that take place during this phase:

**Incident Review**
- Understanding the incident's entire lifecycle, from detection to containment, eradication, and recovery. The team examines the effectiveness of the response actions taken, identifies any gaps or weaknesses in the incident response process, and evaluates the incident's impact on the organization.

**Documentation**
- Documenting the incident's timeline, forensic evidence, response actions, problems encountered, and any other information surrounding the incident is critical. Having the information and having it in an orderly fashion allows the team to analyze the overall incident response process and review its shortcomings. Lack of documentation may lead to the incident occurring once again if nobody ever keeps a historical record of the vent. Proper documentation will be needed when presenting to executives, key stakeholders, and legal administration. 
  
**Lessons Learned/Improvement Plan**
- To tie it all together, the team concludes with what the shortcomings were in the incident response process, and what measures can be taken to strengthen the network posture from outside a technicality level. Improving the security response plan, updating playbooks, conducting more proactive security assessments, and educating end users with an effective cybersecurity program are all methods to help prevent the incident from happening again. 

<br/>
<br/>

**To see the technical process of implementing firewall security controls on the Azure network, see:**

[Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab-Firewall-Hardening)

**To see the effects of implementing these security controls on the Azure network and an overview of the Honeynet/SOC lab project, see:**

[Azure-Cloud-Honeynet-SOC-Lab](https://github.com/gervguerrero/Azure-Cloud-Honeynet-SOC-Lab/tree/main).
