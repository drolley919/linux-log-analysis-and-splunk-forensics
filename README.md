# Linux Log Analysis with Splunk and Password Hash Cracking

## Overview
This project documents a Linux-focused forensic analysis workflow using **Splunk** to review indexed log data and **John the Ripper** to crack password hashes recovered during the investigation. The analysis was performed in a controlled DFCS 635 lab environment and focused on identifying suspicious authentication activity, off-hours logins, software installation events, `chroot` activity, and evidence of weak password hygiene.

The project demonstrates how log aggregation and search can be used to identify indicators of compromise or suspicious administrative behavior, while also showing how recovered credential material can be validated through offline password cracking techniques.

## Objectives
The goals of this project were to:

- Query indexed Linux log data in Splunk for suspicious or security-relevant events
- Identify authentication failures and off-hours login activity
- Investigate software installation activity through package management logs
- Review `chroot`-related events for potentially unusual or security-relevant behavior
- Search for signs of SSH key misuse or permission-related errors
- Crack password hashes using **John the Ripper**
- Document findings, evidence, and forensic conclusions in a structured format suitable for a portfolio case study

## Environment
- **Platform:** Kali Linux forensic lab environment
- **Primary Analysis Tool:** Splunk Enterprise
- **Password Cracking Tool:** John the Ripper
- **Data Sources:** Indexed Linux log files in `/splunk_data/`
- **Hash Format:** Raw-SHA256
- **Course Context:** DFCS 635 – Linux Forensics and Security

## Key Analysis Areas
The investigation focused on the following areas:

### 1. Authentication Failures
Splunk searches were used to identify failed authentication attempts in the indexed Linux log data. These searches helped isolate evidence of unsuccessful login activity and related PAM authentication errors.

### 2. Off-Hours Login Activity
Searches were performed to identify login activity occurring outside expected business hours. This helped highlight potentially suspicious access patterns and allowed review of login timing across the environment.

### 3. SSH / Permission Error Review
Targeted searches were used to look for `Permission denied` activity associated with `id_rsa` or SSH-related authentication issues. Even negative search results were documented because they help rule out certain attack paths or misconfigurations.

### 4. Software Installation Activity
Searches for `apt-get install` were performed to identify software installation events in the environment. These events can be useful for identifying unauthorized package installation, post-compromise tooling, or administrative changes to the system.

### 5. `chroot` Activity
Splunk searches for `chroot` were conducted to identify processes or services invoking a changed root environment. While `chroot` can be benign, it may also be relevant in investigations involving service isolation, application sandboxing, or attempts to obscure execution behavior.

### 6. Password Hash Cracking
Recovered password hashes were processed with **John the Ripper** using targeted wordlists. The objective was to determine whether weak or guessable passwords were present and to validate credential exposure risk through successful password recovery.

## Repository Structure
This repository is organized into the following markdown files:

- **README.md** – Project overview, scope, tools, and repository structure
- **Executive-Summary.md** – High-level summary of the investigation, findings, and outcomes
- **Analysis.md** – Step-by-step explanation of the forensic workflow and interpretation of the results
- **Evidence.md** – Key artifacts, command outputs, and screenshot references supporting the investigation
- **Findings.md** – Consolidated forensic findings identified during log review and password cracking
- **Recommendations.md** – Security recommendations based on the observed activity and recovered credentials

## Screenshots / Evidence Files
The repository includes screenshots documenting Splunk searches, event results, and password cracking output. The evidence set for this project includes:

1. `01-query-authentication-failure-search.png`
2. `02-query-off-hours-login-search.png`
3. `03-query-off-hours-login-results.png`
4. `04-query-authentication-failure-results-overview.png`
5. `05-query-authentication-failure-event-detail.png`
6. `06-query-permission-denied-id-rsa-no-results.png`
7. `07-query-apt-get-install-results-overview.png`
8. `08-query-apt-get-install-event-details.png`
9. `09-query-chroot-search-results.png`
10. `10-query-chroot-event-details.png`
11. `11-john-student-list-crack-results.png`
12. `12-john-show-cracked-passwords-output.png`

## Skills Demonstrated
This project highlights practical skills in:

- Linux log analysis
- Splunk query development and event review
- Authentication and access analysis
- Timeline-oriented event investigation
- Software installation and system activity review
- Password hash cracking and credential analysis
- Digital evidence documentation
- Forensic reporting and portfolio presentation

## Summary
This project demonstrates a structured Linux forensic workflow that combines **log analysis**, **event triage**, and **credential recovery techniques** to investigate suspicious activity in a lab environment. By correlating authentication events, login timing, package installation activity, and password cracking results, the project shows how multiple forensic techniques can be combined to support investigative conclusions and security recommendations.
