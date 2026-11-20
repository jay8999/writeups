# OWASP Top 10:2025 - AS02 — Security Misconfiguration

## 1. Reconnaissance & Enumeration

During the initial assessment of the application and its surrounding infrastructure, the goal was to identify unsafe defaults, incomplete settings, exposed services, and permission flaws across environments, software stacks, and cloud resources.

* **Identifying Exposed Services, Endpoints, & Admin Interfaces:** Focused on discovering publicly reachable administrative panels, debug or monitoring endpoints, unnecessary network services, and unauthenticated API or AI/ML endpoints exposed to the internet.
* **Auditing Cloud Storage & IAM Permissions:** Checked cloud resources (such as AWS S3 buckets, Azure Blob storage, and GCP buckets) and identity roles for overly broad access policies, public read/write permissions, or missing authentication and authorization controls.
* **Inspecting Default Credentials & Outdated Software:** Tested for unchanged default credentials, weak service passwords, and outdated frameworks, servers, or container images with known vulnerabilities or unpatched weaknesses.
* **External Testing:** Validated findings by probing external endpoints, analyzing verbose error responses or stack traces for internal system paths and technology versions, and confirming whether restricted data or functions could be accessed without proper authorization.

## 2. Exploitation

Once a configuration weakness—such as a public storage bucket, exposed administrative interface, default credentials, or verbose error handling—was identified, it was leveraged to access sensitive data or gain unauthorized control:

* **Crafting the Attack Payload:** Prepared targeted requests, scripts, or scanning tools to query exposed endpoints, probe cloud storage URLs, test default credential pairs, or trigger verbose error states.
* **Exploiting Exposed Services & Verbose Errors:** Used stack traces, directory listings, or exposed service banners to map internal architecture and software versions, then accessed unauthenticated administrative consoles, internal APIs, or unprotected AI/ML endpoints.
* **Abusing Misconfigured Permissions & Defaults:** Accessed publicly exposed cloud buckets or overly permissive roles to read, modify, or exfiltrate sensitive records without needing valid user credentials or complex code exploits.
* **Gaining Unauthorized Access:** Retrieved sensitive user or operational data, escalated privileges via misconfigured roles or default accounts, and established a foothold within the environment.

## 3. Remediation

To effectively prevent security misconfigurations and keep environments hardened across their lifecycle, engineering and operations teams should implement the following controls:

* **Harden Configurations & Remove Unused Features:** Disable or remove unnecessary services, endpoints, default accounts, and features, and replace all default or weak credentials before deployment.
* **Enforce Least Privilege & Strong Access Controls:** Lock down cloud storage buckets, APIs, AI/ML endpoints, and internal services with strict authentication, authorization, and least-privilege policies, while segmenting sensitive network resources.
* **Suppress Verbose Errors & Keep Software Patched:** Configure applications to return generic error messages without stack traces or internal system details, and keep operating systems, frameworks, libraries, and containers updated with security patches.
* **Automate Configuration Audits & Pipeline Checks:** Regularly audit cloud configurations and permissions, and integrate configuration reviews and automated security checks into CI/CD deployment pipelines to catch drift early.

---

## Examples

### Example 1: Publicly Accessible Cloud Storage Bucket Exposing Sensitive Data

* **Reconnaissance & Enumeration:** During external infrastructure enumeration, an unauthenticated storage bucket URL is discovered in client or backup configurations. Direct HTTP requests to objects within the bucket succeed without credentials, revealing that access permissions allow public read access.
* **Exploitation:** The attacker queries and downloads sensitive files—such as user records, backups, or operational logs—directly from the misconfigured bucket using standard HTTP clients or cloud CLI tools, bypassing application-level access controls entirely.
* **Remediation:** Block public access by default on all storage buckets, enforce strict IAM and bucket policies using least privilege, and run automated cloud posture checks to detect public exposure prior to release.

### Example 2: Exposed Administrative & Unprotected API Endpoints

* **Reconnaissance & Enumeration:** Endpoint discovery and banner/error inspection reveal an administrative interface or internal API route (including automation or AI/ML endpoints) reachable from the public internet without authentication enforcement, alongside verbose error messages leaking route and service details.
* **Exploitation:** The attacker sends direct requests to the exposed management or API endpoints to view sensitive system configurations, trigger privileged actions, or query underlying data without authenticating as an administrator.
* **Remediation:** Restrict administrative interfaces and sensitive internal endpoints behind VPNs, private networks, or strong authentication and role-based access controls, disable unused routes, and suppress detailed error output in production.

### Example 3: Default Credentials & Outdated Stack Components

* **Reconnaissance & Enumeration:** Service fingerprinting and version checks identify a deployed management service or middleware component running an outdated build with known issues and default administrative credentials still active.
* **Exploitation:** The attacker logs in using published default credentials or leverages known weaknesses in the unpatched component to access administrative controls, inspect environment variables, and pivot toward internal systems.
* **Remediation:** Enforce unique, strong credentials on all accounts and services at provisioning, establish a regular patch management cadence for frameworks, servers, and containers, and verify baseline hardening before promoting builds to production.

