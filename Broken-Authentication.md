# 1. Reconnaissance & Enumeration
During the initial assessment of the web application, the goal was to identify endpoints and mechanisms susceptible to authentication and session management failures.

Identifying Authentication & Registration Endpoints: Focused on sensitive pathways such as login portals, registration pages, password recovery flows, and API endpoints handling credential submission or session generation. These are primary targets for enumeration and brute-force attacks.

Analyzing Rate Limiting & Account Lockout: Checked whether the application enforced restrictions on repeated failed login attempts. Tested if an attacker could perform username enumeration or execute automated credential stuffing attacks without triggering account lockouts, CAPTCHAs, or IP throttling.

Inspecting Session & Cookie Handling: Examined how session identifiers and authentication tokens were generated, transmitted, and stored. Inspected cookies for missing security flags (such as HttpOnly, Secure, and SameSite) and checked whether session IDs were predictable or inadequately randomized.

External Testing: Validated findings by analyzing application responses (such as differing error messages or response times) to determine if valid usernames could be reliably differentiated from invalid ones, enabling targeted credential attacks.

# 2. Exploitation
Once an authentication or session vulnerability—such as weak password policies, lack of rate limiting, or flawed recovery logic—was identified, the weakness was leveraged to compromise user accounts:

Crafting the Attack Payload: Developed scripts or utilized automated tooling to target vulnerable endpoints, such as running high-speed dictionary attacks against login forms lacking rate-limiting controls.

Exploiting Logic Flaws & Enumeration: Leveraged distinct error messages or response behaviors to harvest valid usernames, subsequently narrowing down brute-force efforts to active accounts using common or leaked passwords.

Bypassing Multi-Factor or Session Controls: Where applicable, exploited flaws in multi-factor authentication (MFA) step-up logic or session fixation vulnerabilities to hijack active user sessions without needing valid credentials.

Gaining Unauthorized Access: Successfully authenticated as target users or administrative accounts, achieving unauthorized access to sensitive data, internal system functions, and user-specific controls.

# 3. Remediation
To effectively patch identification and authentication failures and secure user sessions, developers should implement the following defensive practices:

Enforce Multi-Factor Authentication (MFA): Implement robust MFA across the application to prevent automated credential stuffing and brute-force attacks from successfully compromising accounts even if passwords are leaked.

Implement Rate Limiting and Account Lockout: Protect authentication, API login, and password recovery endpoints by implementing strict rate limits and temporary account lockouts (or CAPTCHAs) after a threshold of failed attempts to thwart brute-force and credential stuffing.

Secure Password Policies & Credential Checking: Require strong passwords, check newly created or updated passwords against lists of commonly breached credentials (such as HaveIBeenPwned), and avoid overly restrictive password complexity rules that hinder user creation.

Secure Session and Cookie Management: Ensure that session tokens are cryptographically strong, properly expired upon logout, and transmitted using secure cookies configured with the HttpOnly, Secure, and appropriate SameSite attributes.
