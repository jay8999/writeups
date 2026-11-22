## Vulnerability Writeup: Cryptographic Failures (AS04)

Below is a high level overview of how cryptographic failures are identified, abused conceptually, and remediated, along with illustrative scenarios.

### 1. Reconnaissance & Enumeration

During the initial assessment of the application, the goal was to identify weaknesses where sensitive data or cryptographic controls are improperly protected:

* **Identifying Sensitive Data & Secrets:** Focused on areas handling confidential data, credentials, session tokens, API keys, or model parameters that require confidentiality and integrity.
* **Analyzing Algorithms & Modes:** Checked whether cryptographic operations relied on weak or deprecated algorithms (such as MD5, SHA-1, or ECB mode) instead of modern authenticated encryption.
* **Inspecting Key & Secret Management:** Examined configuration files, repositories, and runtime environments for hard-coded secrets, static keys, or missing key rotation controls.
* **Transport & Storage Assessment:** Evaluated whether data in transit used strong, valid TLS configurations and whether sensitive data at rest was encrypted properly.

### 2. Exploitation

Once a cryptographic weakness was identified, an adversary could leverage it conceptually to compromise confidentiality or integrity:

* **Leveraging Weak or Missing Controls:** Targeted unencrypted channels or weak cryptographic primitives where key material or plain-text values could be intercepted, guessed, or reversed through analytical or brute-force weaknesses.
* **Abusing Exposed Secrets:** Used discovered keys, tokens, or credentials from insecure storage or configuration to bypass access boundaries or decrypt protected data.
* **Triggering Impact:** Intercepted or recovered sensitive information during transit or storage access.
* **Gaining Unauthorized Access:** Used exposed credentials, tokens, or sensitive records to access restricted accounts or internal systems.

### 3. Remediation

To remediate cryptographic failures and protect sensitive data across the application lifecycle, teams should apply the following controls:

* **Use Modern Cryptographic Algorithms:** Adopt strong, modern standards such as AES-GCM or ChaCha20-Poly1305 for encryption, and enforce TLS 1.3 with valid certificates for data in transit.
* **Centralize Key & Secret Management:** Store keys and secrets in dedicated secret management services (such as AWS KMS, Azure Key Vault, or HashiCorp Vault) rather than hard-coding them in source code or configuration files.
* **Enforce Key Rotation & Lifecycle Policies:** Rotate cryptographic keys and secrets on defined schedules, maintain a clear inventory of keys and certificates, and document lifecycle procedures.
* **Protect AI/ML & Automation Workflows:** Ensure AI models, pipelines, and automation agents do not expose unencrypted secrets, credentials, or sensitive inputs.

---

### Examples

#### Example 1: Hard-Coded Secrets & Weak Hashing in Credential Storage

1. **Reconnaissance & Enumeration**
2. **Identifying Sensitive Data & Secrets:** Reviewed authentication and configuration handling where user credentials or internal service keys were processed.
3. **Analyzing Algorithms & Key Management:** Found reliance on fast, weak hashing primitives without proper salting/key stretching or hard-coded secret values embedded in application configuration.
4. **External Testing:** Verified conceptually that weak key derivation or exposed configuration secrets undermined credential and token protection.
5. **Exploitation**
6. **Leveraging Weak Controls:** Evaluated how weak hashing or exposed static secrets lowers resistance to offline guessing or unauthorized token generation.
7. **Abusing Exposed Secrets:** Considered how recovered secret material or weak hashes could allow unauthorized access to protected accounts or services.
8. **Triggering Impact & Gaining Control:** Compromised secret material enables unauthorized authentication or impersonation if left unrotated and unprotected.
9. **Remediation**
10. Replace weak algorithms and hard-coded secrets with strong modern cryptography and a centralized secret manager.
11. Enforce regular key and secret rotation alongside strict access controls.

#### Example 2: Unencrypted or Weakly Protected Sensitive Data in Transit

1. **Reconnaissance & Enumeration**
2. **Identifying Sensitive Data:** Focused on endpoints transmitting authentication tokens, personal data, or sensitive API payloads.
3. **Analyzing Transport Security:** Checked TLS configuration and certificate validation to identify fallback to outdated protocols, invalid/self-signed certificates, or unencrypted channels.
4. **External Testing:** Confirmed through connection inspection whether weak cipher suites or missing transport encryption left traffic exposed.
5. **Exploitation**
6. **Leveraging Weak Controls:** Assessed how lack of strong transport encryption or proper certificate validation exposes data on untrusted networks.
7. **Abusing Exposed Data:** Intercepted or altered traffic could expose session tokens or sensitive payloads in transit.
8. **Triggering Impact & Gaining Control:** Exposed session tokens or credentials can lead to unauthorized session access or data exposure.
9. **Remediation**
10. Enforce TLS 1.3 with strong cipher suites and valid, properly scoped certificates across all endpoints.
11. Maintain a certificate inventory and automate renewal and rotation before expiration.

#### Example 3: Improper Secret Handling in Automated / AI Workflows

1. **Reconnaissance & Enumeration**
2. **Identifying Sensitive Data:** Inspected automated pipelines, integrations, and AI/ML workflows handling API keys, prompts, or sensitive datasets.
3. **Analyzing Key & Secret Management:** Identified risks where secrets or sensitive inputs were passed in plain text, logged, or stored without proper encryption at rest.
4. **External Testing:** Reviewed data flows to confirm whether sensitive values were exposed in logs, configurations, or unencrypted storage.
5. **Exploitation**
6. **Leveraging Weak Controls:** Evaluated how unencrypted storage or verbose logging can leak secrets or sensitive data to unauthorized viewers.
7. **Abusing Exposed Secrets:** Leaked API keys or credentials can be used to query downstream services or access restricted resources.
8. **Triggering Impact & Gaining Control:** Exposed keys or sensitive data can result in unauthorized API usage or data leakage.
9. **Remediation**
10. Encrypt sensitive data at rest with strong algorithms (such as AES-GCM or ChaCha20-Poly1305) and inject secrets securely at runtime via a vault.
11. Redact secrets from logs and enforce lifecycle policies across all automated and AI components.
