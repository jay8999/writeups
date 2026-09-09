# Enumeration & Brute Force

# Reconnaissance & Enumeration

Target Assessment: Discovered open ports and services associated with web application authentication mechanisms.

Error-Based Information Disclosure: Leveraged invalid login attempts, path traversal sequences, and form manipulation to trigger verbose error messages, revealing backend application logic, active usernames, and internal directory structures.

Authentication Analysis: Evaluated HTTP Basic Authentication (RFC 7617), noting that credentials are transmitted as base64-encoded strings within the Authorization header and are vulnerable to interception or brute-force attacks if transmitted over non-HTTPS connections.

# Exploitation

Credential Brute-Forcing: Deployed automated tooling (such as Burp Suite Intruder) and customized wordlists to target authentication portals while managing request parameters to evade detection mechanisms.

Password Reset Weaknesses: Analyzed alternative flows—including email-based tokens, security questions susceptible to PII harvesting, and SMS-based codes vulnerable to interception—to bypass or reset account credentials.

# Remediation

Secure Transport & Hashing: Enforce HTTPS for all basic authentication requests and implement robust session-management tokens (such as OAuth) instead of relying purely on static credentials.

Rate Limiting & Account Lockout: Implement strict throttling, rate limiting, and account lockout policies to neutralize high-speed brute-force attacks and credential stuffing.

Generic Error Messages: Standardize authentication failure messages to prevent user enumeration via differential error responses.
