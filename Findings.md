# Findings

## Overview
This investigation examined Linux log data indexed in **Splunk Enterprise** and password hash cracking activity performed with **John the Ripper**. The goal was to identify evidence of authentication issues, suspicious access timing, system modification activity, environment manipulation, and weak credentials that could affect the security posture of the system.

The findings below summarize the most relevant forensic results derived from the Splunk searches and password recovery process.

---

## Finding 1: Authentication Failure Event Was Identified
A Splunk search for the phrase **"authentication failure"** returned a log entry documenting a failed PAM authentication event. The result showed an **`[ERROR] pam_authenticate failed`** message tied to a Linux host and a syslog source.

### Significance
This finding confirms that at least one unsuccessful authentication attempt occurred on the system and was preserved in the indexed logs. Authentication failures are important because they may indicate:

- An end user entering invalid credentials
- An unauthorized access attempt
- Brute-force or password guessing activity
- Misconfigured authentication processes or service accounts

While a single failed authentication event does not prove malicious behavior on its own, it is still a meaningful indicator that should be preserved within the investigative timeline and correlated with surrounding login or account activity.

---

## Finding 2: Login Activity Was Present Outside Normal Business Hours
A time-filtered Splunk query was used to search for **login-related activity** during the previous seven days while isolating events that occurred **before 09:00 or after 18:00**. This search returned multiple results, demonstrating that login events occurred during non-standard working hours.

### Significance
Login activity outside normal business hours does not automatically indicate compromise, but it is a notable investigative lead because it may reflect:

- Legitimate after-hours administrative access
- Remote access by a user outside normal work schedules
- Suspicious or unauthorized access occurring when oversight is reduced
- Activity related to system maintenance, testing, or malware execution

From a forensic standpoint, off-hours logins should be reviewed alongside user account context, source host details, and other surrounding events to determine whether the access pattern is expected or anomalous.

---

## Finding 3: No Evidence of `id_rsa` Permission-Denied Events Was Found
A Splunk search for **"Permission denied"** combined with **"id_rsa"** returned **no results**. This indicates that the indexed log data reviewed during the investigation did not contain evidence of SSH private key permission errors or failed access attempts involving an `id_rsa` file.

### Significance
This was a negative finding, but it is still important to document because it narrows the scope of suspicious activity. Based on the available data, there was no immediate evidence of:

- SSH failures tied to an improperly secured private key
- Permission problems involving `id_rsa`
- Access attempts that generated those specific log artifacts during the review period

Negative findings help define what was and was not present in the dataset and prevent overstatement of the evidence.

---

## Finding 4: Software Installation Activity Was Recorded in the Logs
A Splunk search for **`apt-get install`** returned multiple software installation events from Linux package management history logs. The results showed installation activity associated with packages such as:

- `waagent`
- `gdebi-core`
- multiple development libraries and supporting packages
- a reinstallation of **Maltego**

### Significance
This finding confirms that package installation activity was preserved in the indexed logs and could be reconstructed during the investigation. Software installation events are valuable because they can reveal:

- Administrative maintenance or system setup actions
- Introduction of new tools or utilities to the system
- Software changes that may coincide with suspicious behavior
- Potential staging of offensive or reconnaissance tooling

The presence of a **Maltego reinstallation** is particularly noteworthy in a forensic context because Maltego is a reconnaissance and link-analysis platform often used for investigative or intelligence work. Its presence does not automatically imply malicious activity, but it is an application worth documenting because it may help explain user behavior, administrative actions, or investigative tooling present on the system.

---

## Finding 5: `chroot` Activity Was Logged Repeatedly
A Splunk search for **`chroot`** returned a substantial number of results. The visible log entries showed repeated messages indicating that an **`rtkit-daemon`** process had **successfully called `chroot`**, with events appearing across multiple dates.

### Significance
`chroot` changes the apparent root directory for a process and is commonly used for containment, service isolation, sandboxing, or controlled execution environments. The presence of repeated `chroot` activity is not inherently malicious, but it is still relevant because it may reflect:

- Normal service behavior or daemon isolation
- Sandboxed execution of processes
- Environment changes performed by system components
- Activity that should be understood in context if privilege abuse or process tampering is suspected

Because these events were tied to a known daemon process rather than an obviously unauthorized shell command, the logs do not by themselves prove malicious behavior. However, they do document system-level activity that could be relevant if the investigation later expands into privilege escalation, service manipulation, or persistence analysis.

---

## Finding 6: Two Password Hashes Were Successfully Cracked
Password recovery testing was performed using **John the Ripper** against a file containing **Raw-SHA256** password hashes. Through dictionary-based cracking and validation with the `--show` option, **two passwords were successfully recovered**:

- **student1 : student2024**
- **nonna : nanan862**

### Significance
This is one of the most important findings in the project because it demonstrates that account credentials protected by the recovered hashes were weak enough to be cracked using common wordlist-based techniques. This creates a meaningful security risk because:

- Attackers who obtain password hashes may be able to recover plaintext credentials
- Weak passwords can enable account compromise, lateral movement, or privilege escalation
- Predictable naming conventions and simple password patterns reduce resistance to offline cracking
- Recovered credentials may overlap with passwords used on other systems or services

The recovery of **student2024** is especially notable because it follows a highly predictable pattern based on a role or account name combined with a year value. Passwords structured this way are vulnerable to dictionary and rules-based attacks.

---

## Finding 7: Password Cracking Workflow Confirmed Full Recovery of the Test Hash Set
The cracking workflow showed that the hash file was processed with different wordlists and validated with the **John `--show`** command after successful cracking. The final output confirmed that **all hashes in the sample file were cracked** and that **zero remained unresolved**.

### Significance
This finding confirms not only that individual passwords were weak, but that the entire scoped set of hashes in the exercise was recoverable with straightforward cracking techniques. In a real-world environment, that would indicate:

- Poor password hygiene among affected users
- Insufficient complexity to resist offline attack
- Increased risk if hashes were extracted from a compromised host
- The need for stronger password policy and user education

This also reinforces the value of combining **forensic log review** with **credential assessment** when evaluating the overall security posture of a Linux system.

---

## Overall Assessment
Taken together, the findings indicate a system with several meaningful forensic artifacts:

- Evidence of at least one **authentication failure**
- **Login activity outside normal business hours**
- Logged **software installation activity**
- Repeated **`chroot`-related system activity**
- **Weak credentials** that were successfully recovered through offline cracking

Not every result indicated malicious behavior, and some findings may reflect normal administrative or system activity. However, the combination of authentication anomalies, off-hours access, software installation evidence, and successfully cracked credentials demonstrates why Linux log analysis and password auditing are both important components of a forensic investigation.

From a defensive perspective, the most significant risk identified in this project was the presence of **recoverable user passwords**, which would materially increase the impact of any compromise involving password hash theft or credential exposure.
