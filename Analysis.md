# Analysis

## Overview
This analysis examines Linux log data ingested into **Splunk Enterprise** and password hash recovery performed with **John the Ripper**. The objective of the project was to determine whether the available logs and credential artifacts revealed suspicious authentication activity, unusual access behavior, system modification events, or weak passwords that could expose the environment to compromise.

The investigation relied on targeted Splunk keyword searches, time-based filtering, and review of package management and daemon activity. The second half of the analysis focused on password cracking using **Raw-SHA256** hashes to determine whether the sample credentials could be recovered through dictionary-based methods.

---

## Authentication Failure Analysis
One of the first tasks in the investigation was to determine whether the log set contained evidence of failed authentication attempts. A Splunk search for **"authentication failure"** returned a single event associated with a **PAM authentication failure**. The event showed an error tied to a syslog source on the Kali system, confirming that at least one login attempt failed.

From an analytical standpoint, authentication failures are important because they often serve as an early indicator of suspicious behavior. They may represent a user entering an incorrect password, an automated brute-force attempt, or an unauthorized user attempting to gain access. In this case, the event count was limited, so the log entry alone does not indicate a large-scale attack or repeated credential guessing. However, the presence of a PAM failure is still significant because it establishes that the authentication process generated a recognizable error event in the dataset and provides a useful pivot point for correlating related login activity.

The authentication failure event should therefore be interpreted as a **meaningful but limited anomaly**. It does not independently prove malicious access, but it does demonstrate that the logs contain failed login evidence that would be valuable in a broader timeline reconstruction.

---

## Off-Hours Login Analysis
The next analytical objective was to determine whether login activity occurred outside normal business hours. To do this, Splunk was used to search for **login events over the previous seven days**, followed by a time-based filter that isolated events occurring **before 09:00 or after 18:00**. This query returned multiple events, showing that login activity did occur during non-standard working hours.

This result is important because time-of-day analysis helps distinguish ordinary activity from potentially suspicious access. In many investigations, after-hours logins are a useful lead because they may indicate unauthorized access, use of a system when legitimate oversight is low, or administrative activity that warrants closer review. At the same time, off-hours logins are not inherently malicious. Administrators, lab users, remote users, or automated services may generate valid access events during evenings or weekends.

Based on the available evidence, the off-hours login results should be interpreted as **notable contextual activity rather than confirmed malicious behavior**. The logs show that the system experienced access-related events outside standard daytime hours, which is enough to justify additional scrutiny, but not enough by itself to conclude that the host was compromised.

---

## SSH / Private Key Error Analysis
To determine whether the logs showed signs of SSH access problems or private key misuse, a search was performed for **"Permission denied"** and **"id_rsa"**. This query returned **zero results**.

Negative results are still important in a forensic investigation because they help define the scope of what is not present in the evidence. In this case, the lack of results suggests that the reviewed logs did not contain obvious evidence of failed SSH authentication attempts or file permission issues involving an `id_rsa` private key. That does not prove SSH was never used, nor does it eliminate the possibility of unlogged or deleted artifacts, but it does reduce the likelihood that private-key access issues were a major component of the suspicious activity visible in the indexed log set.

This portion of the analysis therefore supports a **negative finding**: there was no direct log evidence within the reviewed dataset showing `id_rsa` permission errors or SSH failures tied to that artifact.

---

## Software Installation Activity Analysis
A Splunk search for **`apt-get install`** was used to review software installation history preserved in the package management logs. This query returned multiple events, including package installation or reinstallation activity involving:

- **Maltego**
- **waagent**
- **gdebi-core**
- development libraries and supporting packages such as `curl`, `gcc`, and related dependencies

This was one of the more valuable portions of the investigation because package management activity can reveal administrative behavior, software staging, tool deployment, or post-compromise actions. Forensic analysts often review installation history to determine whether the system was modified around the same time as suspicious events, whether offensive tools were introduced, or whether the system was being actively maintained.

The **Maltego reinstallation** is particularly notable because Maltego is a link analysis and reconnaissance tool. Its presence is not inherently malicious and may be completely legitimate in a lab or research setting, but it is still a software event worth documenting because it demonstrates that the host was used to install or reinstall specialized tooling. The presence of `waagent` and other support packages also indicates normal package-management activity that can help contextualize system usage.

From an analytical perspective, the package installation results show that the host experienced **observable software modification activity**, but the logs do not by themselves prove malicious intent. Instead, they establish that changes to the software environment occurred and were recoverable through Splunk review.

---

## `chroot` Activity Analysis
The search for **`chroot`** returned a large number of results tied primarily to an **`rtkit-daemon`** process. Visible events showed repeated messages indicating that the daemon had **successfully called `chroot`**, with entries occurring across multiple dates and sources such as daemon logs and debug logs.

This finding is important because `chroot` changes the apparent root directory of a process and is often associated with sandboxing, service isolation, or restricted execution environments. In some contexts, `chroot` can also appear in malicious workflows if an attacker is attempting to alter process visibility, isolate tooling, or prepare a modified runtime environment.

In this case, however, the visible log context tied the activity to **`rtkit-daemon`**, which strongly suggests the events are more likely associated with normal system or service behavior than with an interactive attacker. The repeated and successful nature of the events, combined with their association with a daemon process, supports the conclusion that the activity was likely **system-driven rather than overtly malicious**.

That said, repeated `chroot` activity still has forensic value. It shows that the logs captured process-level environment changes and that these events can be identified and traced. If the investigation were expanded, the timestamps of the `chroot` events could be correlated with login activity, process execution, or package installations to determine whether they aligned with other suspicious behavior.

---

## Password Cracking Analysis
The second major component of the project focused on password hash cracking using **John the Ripper**. A file containing **Raw-SHA256** password hashes was tested with multiple cracking workflows. The screenshots show the use of a **student wordlist**, followed by validation of the recovered results using the `--show` option. The final output confirmed that two credentials were recovered:

- **student1 : student2024**
- **nonna : nanan862**

This result is highly significant because it demonstrates that the passwords associated with the hashes were weak enough to be recovered through straightforward dictionary-based cracking. In a real-world investigation, recovered credentials can have major implications. If a threat actor obtains password hashes from a host, weak passwords allow them to recover usable credentials offline without needing to interact with the target system further.

The recovered password **student2024** is especially weak from a defensive standpoint because it follows a predictable structure based on an account role or name plus a year. This kind of password is highly susceptible to dictionary attacks, rules-based mutation attacks, and custom wordlist generation. The second password, **nanan862**, is somewhat less obvious but was still weak enough to be cracked using the available wordlist and cracking workflow.

The cracking results therefore support a strong conclusion: **the system’s credential set included passwords that did not provide meaningful resistance to offline cracking**. This is one of the most impactful findings in the project because it demonstrates a clear security weakness rather than merely a suspicious log event.

---

## Combined Forensic Interpretation
When viewed together, the Splunk results and password cracking results provide a more complete picture of the environment than either source alone. The log review identified:

- at least one **authentication failure**
- **off-hours login activity**
- **software installation events**
- repeated **`chroot` activity**

Meanwhile, the credential analysis confirmed that the environment contained **weak, recoverable passwords**. This matters because weak passwords increase the risk associated with suspicious login activity. If an attacker had obtained access to password hashes, the cracked credentials could have been used to authenticate to the system or pivot to related services.

At the same time, the evidence must be interpreted carefully. The logs do not conclusively show a successful intrusion, privilege escalation, or deliberate attacker persistence. Some of the observed activity—such as package installations or `rtkit-daemon` `chroot` calls—may be completely legitimate within the system’s normal operation. Forensic analysis requires distinguishing between **suspicious artifacts**, **administrative activity**, and **confirmed malicious evidence**. In this project, the strongest confirmed security weakness was not a log event, but rather the successful recovery of plaintext passwords from the available hash set.

---

## Conclusion
The analysis showed that the Linux log data contained useful forensic artifacts related to authentication behavior, off-hours access, package installation activity, and daemon-level `chroot` operations. Although not every search produced suspicious results, the dataset provided enough evidence to demonstrate how Splunk can be used to identify and triage potentially important Linux events.

The password-cracking portion of the investigation produced the clearest security finding. Two user credentials were recovered from Raw-SHA256 hashes, showing that weak password selection materially reduced the security of the environment. Taken together, the project demonstrates the value of combining **log analysis** with **credential assessment** during Linux forensic investigations. Logs can reveal what happened on the system, while password recovery can show how easily an attacker could exploit exposed credential material if the host were compromised.
