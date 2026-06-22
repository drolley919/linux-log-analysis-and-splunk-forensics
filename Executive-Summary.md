# Executive Summary

## Case Overview
This project documents a Linux forensic analysis conducted in a DFCS 635 lab environment using **Splunk Enterprise** and **John the Ripper**. The investigation focused on reviewing indexed Linux log data for suspicious authentication behavior, unusual login timing, software installation activity, and `chroot` execution events. In addition to log review, the project included offline password hash cracking to determine whether weak credentials could be recovered from the available evidence.

The objective was not simply to run keyword searches, but to use log analysis and password recovery techniques to identify potentially meaningful security events, validate whether suspicious activity was present, and assess whether the system showed evidence of weak credential practices or administrative actions that would merit further investigation.

## Scope of Analysis
The forensic review focused on five primary areas:

- **Authentication failure activity**
- **Login activity outside normal working hours**
- **Potential SSH / private key permission issues**
- **Software installation history through `apt-get` logs**
- **`chroot`-related activity in Linux system logs**

A secondary objective involved **password hash cracking** using John the Ripper against recovered **Raw-SHA256** hashes to determine whether weak or guessable passwords were in use.

## Summary of Investigative Actions
The investigation began by querying Splunk for the phrase **"authentication failure"** to identify evidence of failed login attempts. This search returned a PAM authentication failure event tied to a Kali system log source, confirming that at least one unsuccessful authentication attempt was present in the dataset.

Additional Splunk queries were then used to review **login activity over the previous seven days**, with a filter applied to isolate events occurring outside typical daytime working hours. These results showed multiple login-related events occurring during evening or late-night time windows, demonstrating how time-based filtering can be used to surface potentially suspicious access behavior for further review.

To evaluate whether the environment showed signs of SSH misuse or key-related access problems, a search for **"Permission denied"** combined with **"id_rsa"** was performed. This search returned no results, indicating that the available logs did not contain evidence of private key permission errors or failed SSH access attempts involving `id_rsa` during the reviewed timeframe.

The investigation then shifted to system modification activity by searching for **`apt-get install`** events. This search returned multiple software installation records, including package installation activity associated with utilities and tools such as `waagent`, `gdebi-core`, development libraries, and a reinstallation of `maltego`. These results were important because package installation history can help identify administrative actions, new tooling introduced to the system, or software changes that may be relevant in a broader incident investigation.

A separate search for **`chroot`** returned numerous events tied to an `rtkit-daemon` process successfully calling `chroot`. While `chroot` usage is not inherently malicious, the results showed that root-environment changes or related process behavior were recorded in the logs and could be reviewed as part of a broader forensic timeline.

## Password Cracking Results
The final phase of the project focused on **password hash cracking**. Using **John the Ripper** with a student-oriented wordlist and later validating output with the `--show` option, two passwords were successfully recovered from the provided Raw-SHA256 hashes:

- **student1 : student2024**
- **nonna : nanan862**

These results demonstrate that both hashes were vulnerable to dictionary-based password cracking and that the associated passwords were weak enough to be recovered using relatively straightforward methods. From a defensive perspective, this is significant because it illustrates how poor password complexity and predictable naming patterns can expose user accounts to compromise if password hashes are obtained by an attacker.

## Key Findings
The investigation produced several notable findings:

1. **Authentication failure evidence was present** in the indexed Linux logs, confirming at least one failed authentication event in the dataset.
2. **Off-hours login activity was identified**, showing that login-related events occurred outside standard business-hour windows and warrant timeline-based review.
3. **No `id_rsa` / permission-denied evidence was identified** in the reviewed logs, reducing the likelihood of SSH private key misuse within the scoped dataset.
4. **Software installation activity was clearly visible** through `apt-get` history logs, including package installations and a Maltego reinstallation event.
5. **Repeated `chroot` events were present** and tied to system processes, showing evidence of environment or process isolation activity recorded in the logs.
6. **Two user passwords were successfully cracked**, demonstrating weak password security and a tangible credential exposure risk.

## Conclusion
This project demonstrates how Linux forensic investigations often require a combination of **log-centric analysis** and **credential-focused validation** to build a more complete understanding of system activity. Splunk searches were effective for identifying authentication failures, unusual login timing, software installation history, and `chroot`-related events, while John the Ripper provided direct evidence that weak password practices existed within the environment.

Taken together, the results show a system with observable authentication anomalies, visible administrative or software installation activity, and at least two weak credentials that could be recovered through offline cracking. While not every search returned evidence of malicious behavior, the project illustrates a realistic investigative workflow for triaging Linux logs, validating suspicious activity, and documenting forensic findings in a structured and defensible manner.
