# Vulnerability Writeup: Insecure Direct Object Reference (IDOR)

Below is an example of how this vulnerability is often exploited and a proposal of a solution.

# 1. Reconnaissance & Enumeration

  Discovery: During the application mapping phase, traffic was intercepted using an interception proxy (like Burp Suite) while logging into a user account (User A, ID: 1001).

  Endpoints Identified: The user dashboard fetches account details using a predictable GET request: GET /api/v1/user/profile?id=1001.

  Vulnerability Hypothesis: The endpoint accepts a numeric user ID parameter without verifying whether the currently authenticated session token actually owns or has permission to view that specific ID.

# 2. Exploitation

  Step 1: Logged into the application using a standard, low privileged user account (User B, ID: 1002).

  Step 2: Navigated to the profile page and captured the request to GET /api/v1/user/profile?id=1002.

  Step 3: Modified the id parameter in the request from 1002 to 1001 and forwarded the request to the server.

  Result: The application returned a 200 OK response containing the private profile data, email address, and billing history of User A, successfully bypassing authorization boundaries.

# 3. Remediation

  Server-Side Access Control Checks: Implement robust authorization logic on the server side to verify that the logged-in user session matches the requested resource owner before returning data.

  Indirect Reference Maps: Replace direct database identifiers (like sequential integers) with unpredictable, cryptographically secure indirect references, such as UUIDs (e.g., id=550e8400-e29b-41d4-a716-446655440000).

  Principle of Least Privilege: Ensure API endpoints enforce role based access control (RBAC) maps consistently across all controller routes.
