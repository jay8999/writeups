# Guided Pentest: Web - Lab Notes & Writeup

# Reconnaissance & Enumeration

The engagement began with systematic network and application enumeration to map the attack surface of the RecruitX platform.

Network Scanning & Port Discovery

An initial comprehensive TCP port scan was executed using Nmap to identify open ports, running services, and potential entry points on the target (MACHINE_IP):

    nmap -sV -sC -p- MACHINE_IP

Key Findings:

    Port 22 (SSH): OpenSSH 9.6p1 (valuable for later system access if valid credentials are recovered).

    Port 80 (HTTP): Apache httpd 2.4.58 running the main RecruitX web application (RecruitX — Home).

    Port 3306 (MySQL): Database service, indicating backend data storage that is potentially vulnerable to injection or data handling flaws.

    Port 8080 (HTTP): Secondary Apache instance serving a default status/test page.

Application Fingerprinting & Tech Stack

To determine the application framework and headers, a manual HTTP header inspection was performed:

curl -I http://MACHINE_IP

    Server Header: Apache/2.4.58 (Ubuntu)

    Session Management: PHPSESSID cookie observed without the httponly flag set (noted as a secondary hardening issue).

    Technology Stack Confirmed: Classic LAMP stack (Linux, Apache, MySQL, PHP).

Directory & Endpoint Enumeration

Using Gobuster alongside a common wordlist and PHP extension filters, hidden directories and files were mapped across the web root:
Bash

gobuster dir -u http://MACHINE_IP -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x php

High-Value Discoveries:

    /admin: Administrative panel (redirects to login; target for privilege escalation).

    /api: Internal API root containing unauthenticated structural hints.

    /reset.php: Password reset workflow page.

    /uploads: Target directory for user-submitted files.

    /profile.php & /dashboard.php: Authenticated user areas requiring session context.

API Surface Discovery

Querying the unauthenticated /api/ endpoint revealed internal route maps, exposing structural layout information prematurely:

    curl http://MACHINE_IP/api/
    # Output: {"endpoints":["\/api\/user","\/api\/jobs","\/api\/applications"]}

# Exploitation

Gaining remote code execution on the underlying server required chaining multiple low-to-medium severity vulnerabilities together rather than relying on a single complex exploit.

Insecure Direct Object Reference (IDOR): Investigated user-specific endpoints where parameter manipulation exposed unauthorized user records and data, demonstrating a lack of proper server-side authorization checks.

Flawed Password Reset Mechanism: Analyzed the password reset workflow and discovered that the reset token was improperly exposed directly within the server's HTTP response. This design flaw allowed for direct account takeover by intercepting the token and changing targeted accounts' passwords.

File Upload Bypass & Remote Code Execution (RCE):

The file upload form initially relied on a client-side accept attribute to restrict file types, which was easily bypassed by modifying or dropping the client-side validation logic.

The server-side validation relied on a blocklist that failed to account for alternative PHP extensions (such as .phtml or .php5).

Uploaded a web shell using an alternative extension allowed execution of system commands, ultimately leading to full remote code execution on the underlying server.

# Remediation

To secure the application and patch the discovered chain of vulnerabilities, the following engineering and configuration changes must be implemented:

Robust Access Control (IDOR Mitigation): Implement strict, server-side session-based authorization checks for all user-specific endpoints to ensure users can only access data and objects they are explicitly permitted to view.

Secure Password Reset Implementation: Never expose password reset tokens in HTTP response bodies or client-side scripts. Utilize cryptographically secure, randomly generated tokens that expire quickly, and deliver them strictly through out-of-band channels (such as verified user email addresses).

Secure File Upload Validation:

Disregard client-side restrictions (accept attributes) as a security control, treating them strictly as user-experience enhancements.

Implement strict server-side allowlists based on approved file extensions and MIME types rather than blocklists.

Store uploaded files outside of the web root or configure the web server to disable script execution within upload directories.
