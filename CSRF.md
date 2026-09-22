# 1. Reconnaissance & Enumeration

## During the initial assessment of the web application, the goal was to identify endpoints susceptible to Cross-Site Request Forgery (CSRF).

Identifying State-Changing Endpoints: Focused on sensitive actions that modify data or user state, such as password changes, email updates, or financial transactions. These are primary targets for CSRF.

Analyzing HTTP Methods: Checked if sensitive operations were improperly executed via GET requests. Operations performed through GET are much easier to exploit using simple links or embedded images.

Inspecting Tokens & Cookies: Examined whether requests included any anti-CSRF tokens. Discovered endpoints that relied solely on session cookies for authentication without checking the request origin or including a unique token.

External Testing: Validated findings by copying captured requests and attempting to reproduce them from an external context to see if the server accepted them without additional verification.

# 2. Exploitation

## Once an insecure state-changing endpoint lacking proper CSRF defenses was identified, the vulnerability was leveraged to manipulate application logic:

Crafting the Payload: Created an external HTML Proof of Concept (PoC) page containing a malicious form or script designed to target the vulnerable endpoint (e.g., an automated password reset or email update request).

Leveraging Automatic Cookie Inclusion: Because web browsers automatically attach session cookies to cross-origin requests destined for the target application, the malicious page did not need to know the session ID itself.

Triggering the Action: When a simulated authenticated victim visited the external HTML page, the browser automatically sent the crafted request along with the victim's active session cookies.

Gaining Unauthorized Control: The web application processed the request assuming it was intentional, successfully executing the state-changing action (such as modifying the user's account settings) without the victim's knowledge.

# 3. Remediation

## To effectively patch CSRF vulnerabilities and secure state-changing endpoints, developers should implement the following defensive practices:

Implement Anti-CSRF Tokens: Use the Synchronizer Token Pattern. Generate a unique, unpredictable, cryptographically strong token for each user session and require it for all state-changing requests.

Configure SameSite Cookie Attributes: Set the SameSite attribute on session cookies to Strict or Lax to prevent browsers from sending cookies along with cross-site requests.

Enforce Proper HTTP Methods: Ensure that sensitive operations use appropriate HTTP methods like POST, PUT, or DELETE rather than GET.

Require Re-Authentication: For ultra-sensitive actions (like changing passwords or transferring funds), require the user to re-enter their current password before the change is processed.
