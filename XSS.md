# Vulnerability Writeup: Cross-Site Scripting (XSS) - Stored

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
