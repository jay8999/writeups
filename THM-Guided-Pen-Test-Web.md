# Guided Pentest: Web - Lab Notes & Writeup

# Reconnaissance & Enumeration

The engagement began with systematic network and application enumeration to map the attack surface.

Network Scanning: Executed an initial Nmap scan against the target to identify open ports, running services, and potential entry points.

Application Mapping: Mapped the web application's structure, endpoints, headers, and overall behavior before attempting any exploitation. This phase focused on identifying interactive features, such as user profile endpoints, password reset functionalities, and file upload forms.

Attack Surface Identification: Pinpointed high-risk functional areas:

User management endpoints vulnerable to object reference manipulation.

A password reset mechanism handling sensitive token generation.

A file upload utility utilizing client-side restrictions and a weak server-side blocklist.

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
