# Threat Detection, Investigation and Malware Response Using Wazuh SIEM and VirusTotal

A practical Security Operations Centre (SOC) capstone project demonstrating threat detection, security alert investigation, threat intelligence enrichment, malware remediation and incident verification. The project uses **Wazuh SIEM** integrated with **VirusTotal** to identify suspicious endpoint activities, investigate potential security threats and automate malware removal on an Ubuntu endpoint.

## Table of Contents

- Project Overview
- Network Topology
- Tools and Technologies
- Configuration Steps
- Results and Findings
- Author

## Project Overview

This project demonstrates the practical implementation of a **Security Operations Centre (SOC) incident management lifecycle** using Wazuh SIEM and VirusTotal.

The primary objective was to monitor endpoint activities, identify suspicious behaviour, investigate security alerts, enrich indicators of compromise (IOCs), implement remediation and verify that identified threats had been successfully removed.

The investigation focused on three main security activities:

- **Privilege Escalation:** Detection and investigation of commands executed with administrative privileges.
- **Suspicious Account Creation:** Identification of a newly created user account and associated file integrity changes.
- **Malware Detection and Response:** Identification of a suspicious file, analysis through VirusTotal and automated removal using Wazuh Active Response.

The project followed the five essential stages of SOC incident management:

1. **Detect:** Identify suspicious activities through Wazuh security alerts.
2. **Investigate:** Analyse endpoint logs, commands and security events.
3. **Enrich:** Use VirusTotal to investigate file hashes and obtain threat intelligence.
4. **Respond:** Implement automated malware removal using a Bash active response script.
5. **Verify:** Review subsequent security events and confirm successful remediation.

**Investigation Period:** 22–29 October 2026.

## Network Topology

The project used an Ubuntu virtual machine monitored by Wazuh SIEM, with VirusTotal providing external threat intelligence enrichment.

### Network Architecture

- **Endpoint:** Ubuntu 2 (Ubuntu-VirtualBox).
- **Operating System:** Ubuntu Linux.
- **SIEM Platform:** Wazuh.
- **Threat Intelligence Platform:** VirusTotal.
- **Active Response:** Bash script integrated with Wazuh.
- **JSON Processing Utility:** jq.

### Network Components

| Component | Description |
|---|---|
| Ubuntu 2 | Endpoint used for security monitoring and investigation |
| Wazuh SIEM | Collected and analysed endpoint security events |
| VirusTotal | Analysed suspicious file hashes and provided threat intelligence |
| Wazuh File Integrity Monitoring | Detected file creation and checksum changes |
| Wazuh Active Response | Executed automated malware remediation |
| jq | Processed JSON input supplied to the response script |

### Key Endpoint Information

- **Hostname:** `ubuntu-VirtualBox`
- **Endpoint:** `Ubuntu 2`
- **User Account:** `ubuntu`
- **User UID:** `1000`
- **Root UID:** `0`
- **Suspicious Account:** `soc_test`
- **Suspicious File:** `/root/eicar.com`
- **Additional File Event:** `/etc/cups/subscriptions.conf.N`

### Security Event Information

| Activity | Wazuh Rule ID | Severity |
|---|---|---|
| Privileged command execution | `5502` | Medium |
| Privileged shell execution | `5502` | Low |
| Suspicious account creation | `5902` | Medium |
| File creation event | `554` | File integrity alert |

**Note:** The report does not specify the IP addresses assigned to the Ubuntu endpoint or Wazuh server. These should be added from the actual lab configuration if available.

## Tools and Technologies

The following tools and technologies were used throughout the project:

- **Wazuh SIEM:** Security monitoring, event collection, alert generation and incident response.
- **VirusTotal:** Threat intelligence enrichment and malware analysis.
- **Ubuntu Linux:** Endpoint operating system used for security testing.
- **Oracle VirtualBox:** Virtualisation environment hosting the Ubuntu endpoint.
- **Bash Scripting:** Development of the automated malware removal script.
- **jq:** JSON processing utility required by the active response script.
- **File Integrity Monitoring (FIM):** Detection of suspicious file creation, modification and deletion.
- **Linux Terminal:** Execution of administrative commands and investigation activities.
- **MITRE ATT&CK Framework:** Classification of observed security activities, including privilege escalation and persistence.
- **SHA-256 and MD5:** Cryptographic hashes used to identify and investigate suspicious files.

## Configuration Steps

### 1. Prepare the Ubuntu Endpoint

The Ubuntu 2 virtual machine was used as the monitored endpoint.

The endpoint was configured to communicate with the Wazuh monitoring environment, allowing security events and system activities to be collected.

### 2. Generate and Detect Privileged Command Execution

The first investigation involved executing a command with administrative privileges.

```bash
sudo id
```

The command displayed the user identity, UID, GID and group memberships.

**Wazuh Detection:**
- Rule ID: `5502`
- Severity: Medium
- Date: 22 October 2026
- Time: 12:42:52

The alert demonstrated successful execution of a privileged command.

### 3. Investigate Privileged Shell Execution

The second activity involved executing multiple system identification commands within a privileged shell.

```bash
sudo sh -c 'id;whoami;hostname'
```

The commands performed the following functions:

- `id`: Displayed user and group information.
- `whoami`: Identified the current user.
- `hostname`: Displayed the endpoint hostname.

**Wazuh Detection:**
- Rule ID: `5502`
- Severity: Low
- Date: 22 October 2026
- Time: 13:21:53

The investigation confirmed that the commands were executed within a root-level session.

### 4. Investigate Suspicious User Account Creation

The third activity involved creating a new Linux user account and modifying its password configuration.

```bash
sudo useradd -m soc_test && sudo passwd -d soc_test
```

The command created a user named `soc_test` and executed a password modification operation.

**Wazuh Detection:**
- Rule ID: `5902`
- Severity: Medium
- Date: 22 October 2026
- Time: 16:02:11

Wazuh generated a file integrity monitoring alert indicating a checksum change.

This activity was investigated because unauthorised account creation can provide attackers with an alternative route to maintain access to a compromised endpoint.

### 5. Detect Suspicious File Activity

On 29 October 2026, Wazuh detected a file creation event on the Ubuntu endpoint.

**Event details:**
- File path: `/etc/cups/subscriptions.conf.N`
- Endpoint: Ubuntu 2
- Rule ID: `554`
- Date: 29 October 2026
- Time: 11:32:50

A separate malware investigation identified the suspicious file `eicar.com` under:

```bash
/root/eicar.com
```

The relationship between the initial file creation event and the file submitted to VirusTotal should be confirmed using endpoint hashes and event records.

### 6. Enrich the Investigation Using VirusTotal

The suspicious file was submitted to VirusTotal for threat intelligence analysis.

The following file hashes were recorded:

**SHA-256:**

```text
275a021bbfb6489e54d471899f7db9d1663fc695ec2fe2a2c4538aabf651fd0f
```

**MD5:**

```text
44d88612fea8a8f36de82e1278abb02f
```

VirusTotal returned the following results:

- Detection status: Malicious.
- Antivirus security engines detecting the file: **64**.
- Threat classification: Trojan.
- Relevant classification: Exploitation for Client Execution.
- Additional indicators: **10 suspicious IP addresses**.

The results provided supporting evidence for classifying the file as malicious and proceeding with remediation.

### 7. Install the jq Utility

The `jq` utility was installed on the Ubuntu endpoint to enable the active response script to process JSON input.

First, update the package repositories:

```bash
sudo apt update
```

Install jq:

```bash
sudo apt -y install jq
```

### 8. Configure the Wazuh Active Response Script

A Bash script named `remove-threat.sh` was created to automate the removal of identified malicious files.

The script was saved in the Wazuh active response directory:

```bash
/var/ossec/active-response/bin/remove-threat.sh
```

The purpose of the script was to process the active response input and remove identified malicious files from the affected endpoint.

The mechanism was integrated with Wazuh to support automated remediation.

### 9. Execute Malware Remediation

The active response mechanism was executed against the identified malicious file.

The documented remediation outcomes included:

- Successful removal of the identified malicious file.
- Wazuh recording the response action.
- Further endpoint file integrity monitoring.
- No subsequent observation of the suspicious file activity.

### 10. Verify the Remediation

The final stage involved reviewing subsequent Wazuh file integrity monitoring events.

The verification process confirmed:

1. The active response script successfully removed the identified malicious file.
2. Wazuh recorded the remediation activity.
3. Subsequent file integrity monitoring showed that the suspicious file was no longer present.
4. The previously identified suspicious file activity was no longer observed.

Further network traffic analysis and endpoint investigation would be required before declaring the wider environment completely free from compromise.

## Results and Findings

The project successfully demonstrated the use of Wazuh SIEM and VirusTotal for endpoint security monitoring, malware investigation and automated incident response.

### 1. Privilege Escalation Detection

Wazuh successfully generated alerts when administrative commands were executed on the Ubuntu endpoint.

The investigation identified successful root-level command execution and system reconnaissance activities.

### 2. Suspicious Account Creation

Wazuh detected the creation of a new user account named `soc_test`.

The associated file integrity monitoring alert demonstrated how changes to system files can provide useful evidence during security investigations.

Further checks would be required to establish whether the account was authorised or added to privileged groups.

### 3. Malware Detection

VirusTotal identified the suspicious file as malicious, with **64 antivirus security engines** detecting it.

The investigation also identified:
- Trojan-related threat classification.
- Exploitation for Client Execution classification.
- 10 suspicious IP addresses requiring further investigation.

### 4. Automated Malware Remediation

The Wazuh Active Response mechanism successfully removed the identified malicious file using the Bash script `remove-threat.sh`.

Wazuh recorded the remediation activity, demonstrating how automated response mechanisms can support faster incident containment.

### 5. Incident Verification

Subsequent file integrity monitoring provided evidence that the suspicious file was no longer present on the endpoint.

The available evidence supported successful remediation of the identified file-based threat.

### Summary of Results

| Security Activity | Result |
|---|---|
| Privileged command detection | Successfully detected |
| Privileged shell investigation | Successfully investigated |
| Suspicious account creation | Detected and investigated |
| File integrity monitoring | File activity detected |
| VirusTotal enrichment | 64 antivirus detections |
| Malware classification | Trojan |
| Suspicious IP indicators | 10 identified |
| Active response configuration | Implemented |
| Malware removal | Successful |
| Remediation verification | Confirmed by available Wazuh evidence |

### Key Learning Outcomes

Through this project, I developed practical experience in:

- Security alert monitoring and triage.
- SIEM event investigation and log analysis.
- Linux privilege escalation detection.
- Suspicious account activity investigation.
- File integrity monitoring.
- Threat intelligence enrichment.
- Malware analysis using VirusTotal.
- IOC identification and investigation.
- Bash scripting and automated incident response.
- Incident remediation and post-incident verification.
- Applying the SOC incident management lifecycle.

### Recommendations

The following improvements were identified:

- Strengthen privileged access management and review `sudo` permissions.
- Configure additional alerts for account creation and password changes.
- Improve file integrity monitoring across sensitive system directories.
- Continue integrating VirusTotal with Wazuh.
- Develop and test additional active response scripts.
- Investigate network traffic associated with the 10 suspicious IP addresses.
- Conduct continued endpoint monitoring to identify recurring indicators of compromise.

### Conclusion

This capstone project demonstrated the practical application of the five-stage SOC incident management lifecycle: **Detect, Investigate, Enrich, Respond and Verify**.

Using Wazuh SIEM, VirusTotal, Ubuntu Linux and Bash scripting, I successfully identified suspicious endpoint activities, investigated potential security threats, enriched malware indicators and implemented automated malware removal.

The project strengthened my practical understanding of SOC operations, threat detection, incident investigation, malware analysis, endpoint monitoring and automated incident response.

**Final Assessment:** The identified malicious file was successfully removed, and the available Wazuh evidence supported successful remediation. Additional endpoint and network investigations would be necessary to confirm that the wider environment was completely free from compromise.

## Author

**Taofeek Abdulkadir**

Cybersecurity Analyst | SOC Analyst 

**Areas of Interest:**
- Security Operations Centre (SOC)
- Security Information and Event Management (SIEM)
- Threat Detection and Incident Response
- Malware Analysis
- Threat Intelligence
- Vulnerability Management
- Linux Security
- Digital Forensics

**Professional Profile:** LinkedIn – www.linkedin.com/in/taofeek-abdulkadir-08a439b4

**GitHub:** https://github.com/taofeekadri-cmd/SOC-CAPSTONE-PROJECT/edit/main/README.md

**Email:** taofeekadri@gmail.com

