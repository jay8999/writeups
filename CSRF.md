# Vulnerability Writeup: Cross Site Request Forgery (CSRF)

Below is an example of how this vulnerability is often exploited and a proposal of a solution.

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

# Examples

## Example 1: Malicious Email Update via an Auto-Submitting Form (Classic POST CSRF)

1. Reconnaissance & Enumeration

    Identifying State-Changing Endpoints: Targeted the account settings page where users update their registered email address (/api/account/update-email).

    Analyzing HTTP Methods & Tokens: Discovered the endpoint accepted standard POST requests containing only the user's session cookie. It lacked any anti-CSRF tokens (such as a synchronizer token or double-submit cookie).

    External Testing: Captured a sample request and confirmed that reproducing it from an external origin successfully updated the email without server-side origin validation.

2. Exploitation

    Crafting the Payload: Created an external HTML Proof of Concept (PoC) page hosting a hidden form pre-populated with the attacker's email address and an automated JavaScript form.submit() function.

    Leveraging Automatic Cookie Inclusion: Because the victim's browser automatically attaches session cookies to cross-origin requests, no prior knowledge of the session ID was required.

    Triggering the Action & Gaining Control: When an authenticated victim visited the malicious site, the form auto-submitted, silently changing the account email to the attacker's address, allowing them to initiate a password reset and fully take over the account.

3. Remediation

    Implement the Synchronizer Token Pattern to require a unique, unpredictable anti-CSRF token for every session on state-changing requests.

    Require re-authentication (entering the current password) before allowing critical profile modifications like email or password changes.

## Example 2: Unauthorized Fund Transfer via a Crafted GET Request

1. Reconnaissance & Enumeration

  Identifying State-Changing Endpoints: Focused on the banking portal's quick-transfer feature (/transfer?amount=500&to_account=98765).

  Analyzing HTTP Methods: Found that the financial transaction was improperly implemented using a GET request instead of a POST or PUT method.

  Inspecting Tokens & Cookies: The endpoint relied entirely on session cookies for authorization and completely ignored anti-CSRF controls.

2. Exploitation

  Crafting the Payload: Built a simple malicious blog post embedding an image tag pointing directly to the vulnerable transfer URL: <img src="[https://bank.com/transfer?amount=500&to_account=98765](https://bank.com/transfer?amount=500&to_account=98765)" width="0"         height="0">.

  Leveraging Automatic Cookie Inclusion: When the victim loaded the blog post, their browser automatically tried to fetch the "image" by sending a GET request to the banking application, attaching their active banking session cookies along with it.

  Gaining Unauthorized Control: The bank processed the request as an intentional transfer from the user, successfully moving funds to the attacker's account without the victim's knowledge or consent.

3. Remediation

  Enforce Proper HTTP Methods: Restrict all state-changing operations (like transfers, purchases, or deletions) to methods that modify state safely (e.g., POST, PUT, or DELETE) rather than GET.

  Configure SameSite=Strict on session cookies so that the browser never sends cookies alongside cross-origin requests originating from external links or images.

## Example 3: Account Deletion via Cross-Origin AJAX Request

1. Reconnaissance & Enumeration

  Identifying State-Changing Endpoints: Investigated the account deactivation feature (/api/v1/user/delete-account).
  
  Inspecting Tokens & Cookies: While the application utilized custom headers, reconnaissance revealed that the CORS (Cross-Origin Resource Sharing) policy was misconfigured (Access-Control-Allow-Origin: *) and the endpoint accepted simple content types without verifying custom token headers.
  
  External Testing: Verified that an external script could successfully invoke a fetch() request to the delete endpoint and force the application to tear down the user account.

2. Exploitation

  Crafting the Payload: Developed a JavaScript payload leveraging a cross-origin fetch request with credentials: 'include' to target the account deletion endpoint.
  
  Triggering the Action: Tricked an authenticated user into visiting a landing page hosting the script.
  
  Gaining Unauthorized Control: The browser fired the background request, attached the victim's valid session cookie under the permissive CORS rules, and permanently deleted the user's account and all associated data.

3. Remediation

  Configure SameSite Cookie Attributes: Set session cookies to SameSite=Lax or Strict to block cross-site cookie transmission on state-changing fetch requests.
  
  Implement Custom Request Headers: Require non-standard headers (e.g., X-Requested-With or custom anti-CSRF bearer tokens) for all API requests, which cannot be forged across origins due to CORS preflight restrictions.
