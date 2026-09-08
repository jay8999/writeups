# Guided Pentest: Infrastructure - Lab Notes & Writeup

# Reconnaissance & Enumeration

The engagement began with an unauthenticated black-box assessment targeting a single unknown IP address, simulating a real-world infrastructure penetration test.

Port & Service Scanning: Executed comprehensive network scans to map out all open ports and running services.

Service Analysis: Evaluated the enumeration results against known vulnerability databases to identify outdated software versions, weak configurations, and potential public exploit vectors.

# Exploitation

Achieving full system compromise required moving from initial access via a public exploit to local privilege escalation through filesystem misconfigurations.

Initial Foothold: Identified a vulnerable service running on the target and leveraged a matching public exploit to gain an initial low-privileged shell.

Local Enumeration: Performed basic post-exploitation enumeration using standard Linux command-line utilities to inspect the filesystem and configuration settings.

Privilege Escalation (Plaintext Credentials):

Discovered a critical misconfiguration: the root user's password was stored in plaintext within a file located at /etc/password.txt.

Verified that the file permissions allowed low-privileged users to read its contents.

Read the password using cat /etc/password.txt and successfully authenticated via SSH (ssh root@IP) to achieve full root system compromise.

# Remediation

To eliminate the critical vulnerabilities exposed during this infrastructure engagement, the following security measures must be implemented:

Immediate Secret Removal & Rotation: Remove the plaintext password file (/etc/password.txt) immediately from the filesystem and rotate the root password.

Secure Credential Storage: Never store credentials in plaintext on disk. Rely instead on native system authentication mechanisms (such as /etc/shadow protected by robust cryptographic hashing) or a dedicated secrets management solution.

Enforce Least Privilege: Audit and strictly restrict file permissions across the system to ensure low-privileged users cannot access sensitive configuration files or administrative artifacts.
