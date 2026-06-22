# Recommendations

## Overview
The investigation identified authentication anomalies, off-hours login activity, package installation events, `chroot`-related log activity, and weak passwords that were successfully cracked from Raw-SHA256 hashes. While not every finding indicates malicious behavior on its own, the combined evidence highlights several areas where security controls, visibility, and credential hygiene should be improved.

The recommendations below are intended to reduce the likelihood of credential compromise, improve detection of suspicious Linux activity, and strengthen the organization’s ability to investigate authentication and system-level events.

---

## 1. Enforce Stronger Password Requirements
The most significant security weakness identified during this project was the successful recovery of two user passwords through offline cracking. Passwords such as **student2024** demonstrate predictable construction patterns that are highly vulnerable to dictionary-based and rules-based attacks.

### Recommended actions
- Require **longer and more complex passwords** for all local and domain accounts
- Prohibit passwords based on usernames, seasons, years, or common keyboard patterns
- Implement password screening against known weak-password lists
- Educate users on how predictable passwords can be recovered from stolen hash data

### Why it matters
If an attacker obtains password hashes from a Linux system, weak passwords can often be cracked offline without interacting with the host. Strong password policies reduce the value of stolen credential material and make password cracking substantially more difficult.

---

## 2. Implement Multi-Factor Authentication Where Possible
Authentication failures and cracked credentials both highlight the risk associated with relying solely on passwords for access control.

### Recommended actions
- Require **multi-factor authentication (MFA)** for administrative access, remote access, and privileged Linux accounts
- Use MFA for VPN, SSH gateways, and web-based administration portals whenever supported
- Apply stronger authentication controls to accounts with elevated privileges or access to sensitive systems

### Why it matters
Even if a password is guessed, stolen, or cracked, MFA provides an additional barrier that can prevent account compromise.

---

## 3. Monitor Authentication Failures More Aggressively
The Splunk analysis identified a PAM authentication failure event. While the project only surfaced a limited number of failures, repeated or clustered authentication failures can be an important indicator of password spraying, brute-force attempts, or user account misuse.

### Recommended actions
- Create Splunk alerts for repeated authentication failures from the same source host, username, or time window
- Establish thresholds for unusual login failures, especially on privileged accounts
- Correlate failed logins with successful logins, password resets, and account lockout events
- Review PAM, SSH, XRDP, and syslog authentication events on a recurring basis

### Why it matters
Authentication failures often appear early in an attack chain. Monitoring them proactively can shorten detection time and reveal compromised or targeted accounts before broader damage occurs.

---

## 4. Baseline and Investigate Off-Hours Login Activity
The investigation identified login-related activity outside standard working hours. While this may be legitimate in some environments, it should still be baselined and monitored because it can indicate unauthorized access or after-hours administrative activity that deserves review.

### Recommended actions
- Define normal working-hour baselines for users, administrators, and service accounts
- Create Splunk dashboards or alerts for **after-hours logins**, especially on critical systems
- Review off-hours access alongside source IPs, usernames, hostnames, and subsequent activity
- Require additional justification or approval for certain types of after-hours privileged access

### Why it matters
Time-based anomalies are often easier to detect than subtle command-level activity. Reviewing access outside normal patterns helps surface behavior that may otherwise be missed.

---

## 5. Maintain Visibility into Package Installation Activity
The `apt-get install` searches showed that package installation history can provide valuable insight into software changes on a Linux host. Software installation events may be benign, but they can also reveal staging of tools, system modification, or persistence-related behavior.

### Recommended actions
- Retain and centralize APT / package management logs for all Linux hosts
- Alert on installation of dual-use or high-risk tools where appropriate
- Correlate package installation events with user logins, sudo activity, and shell history
- Periodically review installed packages for unauthorized software or unsupported tools

### Why it matters
Package-management history is a strong source of forensic evidence. Tracking software changes helps investigators determine what was introduced to a host, when it happened, and which account may have performed the action.

---

## 6. Review `chroot` and Service-Isolation Activity in Context
The Splunk review identified repeated `chroot` events tied to `rtkit-daemon`. These events may be completely legitimate, but repeated service-level environment changes should be understood within the normal behavior of the system.

### Recommended actions
- Baseline which daemons and services normally generate `chroot` or sandbox-related log entries
- Investigate new or unexpected `chroot` activity, especially if it appears with privilege escalation or suspicious process creation
- Retain daemon and debug logs long enough to support retrospective analysis
- Correlate `chroot` events with service restarts, package installations, and configuration changes

### Why it matters
Context matters. A `chroot` event generated by a trusted daemon may be normal, but the same event associated with an unexpected process or account could indicate malicious execution or environment manipulation.

---

## 7. Improve Linux Log Centralization and Retention
This project demonstrates the value of centralized log search in Splunk. Without indexed and searchable logs, authentication anomalies, package installation history, and daemon activity would be more difficult to reconstruct.

### Recommended actions
- Forward authentication logs, syslog, daemon logs, package-management logs, and audit logs to a central SIEM
- Standardize log retention periods for Linux systems based on security and compliance requirements
- Ensure timestamp normalization and host metadata are preserved during ingestion
- Tag logs by host, environment, and role to improve triage and investigation speed

### Why it matters
Centralized logging improves both **security monitoring** and **forensic readiness**. It allows analysts to pivot across hosts, correlate events, and search historical data without relying on a single endpoint’s local logs.

---

## 8. Expand Splunk Detection Content for Linux Systems
The queries used in this project were effective as manual investigative searches, but many of them could be converted into recurring detections or dashboards.

### Recommended actions
Develop Splunk detections or saved searches for:

- Authentication failures
- Off-hours login events
- New software installations
- Repeated `sudo` or privilege escalation attempts
- SSH key permission errors
- Unusual `chroot` activity
- Password spraying or login bursts

### Why it matters
Turning investigative queries into repeatable detections helps move from reactive forensics toward proactive security monitoring.

---

## 9. Protect Password Hashes and Credential Stores
Because password cracking was successful in this project, it is important to reduce the likelihood that attackers can obtain hashes in the first place.

### Recommended actions
- Restrict access to `/etc/shadow`, password databases, backups, and credential dumps
- Limit root and sudo access to only those who require it
- Monitor for attempts to copy, exfiltrate, or access password hash files
- Secure backups and snapshots that may contain credential material
- Rotate passwords immediately if hashes are exposed during an incident

### Why it matters
Even strong passwords become vulnerable if attackers can repeatedly attack hashes offline. Preventing access to hash material is a critical defensive control.

---

## 10. Conduct Periodic Password Auditing in a Controlled Manner
The successful John the Ripper results demonstrate that password auditing can reveal real weaknesses before attackers exploit them.

### Recommended actions
- Perform authorized password auditing in controlled internal security assessments
- Use cracking results to identify weak-password patterns and improve policy enforcement
- Pair password audits with user awareness training and remediation plans
- Document results carefully and handle cracked credentials as sensitive information

### Why it matters
Proactive credential auditing helps organizations find weak passwords before an attacker does. It also provides concrete evidence for improving password policies and user behavior.

---

## Conclusion
This investigation demonstrated how Linux log analysis and password-cracking workflows can reveal both **behavioral anomalies** and **credential-related weaknesses** within an environment. The most important risk identified in this project was the presence of weak passwords that could be recovered through straightforward cracking methods. Additional findings, including authentication failures, off-hours logins, software installation events, and daemon-driven `chroot` activity, highlight the need for strong monitoring, logging, and contextual review of Linux systems.

By implementing stronger password controls, improving Linux log visibility, baselining authentication and package-management activity, and expanding Splunk detections, organizations can reduce the risk of compromise and improve their ability to investigate suspicious events quickly and effectively.
