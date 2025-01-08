# AWS
# Building a Mini Security Environment in AWS (Live Traffic)

AWS Security Environment
![image](https://github.com/user-attachments/assets/f0d430a2-41a1-4130-aa2a-b09abb3f4b5d)


## Introduction

In this project, I build a mini security environment in AWS and ingest log sources from various resources into Amazon CloudWatch Logs. From there, I utilize Amazon GuardDuty and AWS Security Hub to visualize threats, trigger alerts, and generate findings. I measured some security metrics in the insecure environment for 24 hours, applied certain security controls to harden the environment, then measured metrics for another 24 hours. The metrics we will show are:

- **Windows Event Logs** (via CloudWatch Agent)
- **Linux Syslog** (via CloudWatch Agent)
- **GuardDuty Findings** (Alerts triggered within GuardDuty)
- **Security Hub Findings** (Consolidated security alerts)
- **VPC Flow Logs** (Allowed or denied traffic into our environment)

---

## Architecture Before Hardening / Security Controls

![image](https://github.com/user-attachments/assets/226f3a08-a6de-4a0d-b0aa-4a4ed484f9a6)


### Description of the Environment

- **Amazon VPC**: A single VPC with a public subnet.  
- **Security Group (SG)**: Inbound rules wide open (e.g., 0.0.0.0/0 allowed for SSH, RDP, etc.).  
- **EC2 Instances**: 
  - 2 Windows instances  
  - 1 Linux instance  
- **CloudWatch Logs**: Collects system logs from EC2 instances via agents.  
- **S3 Bucket**: Publicly accessible for testing.  
- **AWS Key Management Service (KMS)**: Minimal usage, no strong key policies.  
- **Amazon GuardDuty**: Enabled to detect suspicious activities, but no blocking mechanism in place.

In this **“BEFORE”** state, resources are intentionally exposed to the internet with minimal restrictions, reflecting a common misconfiguration scenario.

---

## Architecture After Hardening / Security Controls

![Architecture Diagram (After)](https://i.imgur.com/fiK12O4.jpg)

### Security Improvements

- **Amazon VPC**: Added private subnets; restricted inbound traffic using NAT Gateway for outbound.  
- **Security Group (SG)**: Inbound rules restricted to specific admin IP addresses.  
- **EC2 Instances**:  
  - Windows and Linux instances placed in private subnets.  
  - Instances only accessible via Bastion Host (jump box).  
- **CloudWatch Agent**: Continues forwarding system logs, but with stricter IAM roles.  
- **S3 Bucket**: Switched to private access with Bucket Policies limiting who can get objects.  
- **AWS KMS**: Configured custom key policies and enforced encryption at rest.  
- **Amazon GuardDuty & Security Hub**: Integrated with AWS Organizations, generating consolidated findings with potential to auto-remediate.

With these improvements, the environment drastically reduces its attack surface, preventing unauthorized traffic and ensuring thorough logging and monitoring.

---

## Attack Maps Before Hardening / Security Controls

Below are some examples of suspicious activity captured before implementing controls:

1. **VPC Flow Logs** – Allowed inbound traffic from malicious IP addresses.  
   ![Allowed Malicious Flows](https://i.imgur.com/pcGApEe.png)

2. **Linux Syslog Auth Failures** – Numerous SSH brute force attempts.  
   ![Linux SSH Auth Failures](https://i.imgur.com/F5osLM2.png)

3. **Windows RDP/SMB Auth Failures** – Repeated RDP and SMB login attempts.  
   ![Windows Auth Failures](https://i.imgur.com/RaBGo5e.png)

---

## Metrics Before Hardening / Security Controls

The following table shows the metrics we measured in our **insecure** AWS environment over 24 hours:  
**Start Time**: 2024-03-10 14:00  
**Stop Time**:  2024-03-11 14:00  

| Metric              | Count  |
| ------------------- | -----: |
| Windows Event Logs  | 16,380 |
| Linux Syslog        | 2,942  |
| GuardDuty Findings  | 8      |
| Security Hub Alerts | 298    |
| VPC Flow Logs       | 1,220  |

> *Note:* Most of the malicious traffic originated from unknown IPs scanning open SSH/RDP ports.

---

## Attack Maps After Hardening / Security Controls

After applying security controls, suspicious inbound traffic was drastically reduced. Any attempted connections from unauthorized IP ranges were blocked at the Security Group or NAT Gateway level. The dashboards now show minimal or no unauthorized access attempts:

- **Allowed Malicious Flows**: Essentially zero from external sources.  
- **Linux Syslog Auth Failures**: Only local or legitimate admin attempts.  
- **Windows RDP/SMB Auth Failures**: No brute force from unknown IP addresses.

---

## Metrics After Hardening / Security Controls

This table shows the metrics measured in our **secured** AWS environment for another 24 hours:  
**Start Time**: 2024-03-12 15:00  
**Stop Time**:  2024-03-13 15:00  

| Metric              | Count |
| ------------------- | ----: |
| Windows Event Logs  | 6,489 |
| Linux Syslog        |  217  |
| GuardDuty Findings  |    0  |
| Security Hub Alerts |    0  |
| VPC Flow Logs       |   180 |

> *Note:* The sharp decrease in logs indicates minimal unauthorized activity. The remaining logs mainly reflect legitimate administrative tasks and routine system events.

---

## Conclusion

In this project, a mini security environment was constructed in AWS, and log sources were forwarded to Amazon CloudWatch. Amazon GuardDuty and Security Hub helped identify threats and produce consolidated alerts. Comparing metrics before and after applying security controls demonstrates:

- A **significant drop** in suspicious flows and login attempts.  
- **Zero** GuardDuty findings or Security Hub alerts once the system was properly hardened.  
- Legitimate traffic remained uninterrupted, while malicious attempts were effectively blocked.

If these AWS resources were more actively used by end users, we might see higher overall log volumes in the hardened environment, but the attack-related events would still be minimal, thanks to stricter security measures.

---
