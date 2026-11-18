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

# Examples

## Example 1: E-Commerce Order History & Shipping Details
1. Reconnaissance & Enumeration

    Discovery: During the application mapping phase, traffic was intercepted using an interception proxy while viewing an order confirmation receipt after purchasing a product.

    Endpoints Identified: The order history portal fetches records using a predictable GET request: GET /api/v1/orders/view?order_id=8831.

    Vulnerability Hypothesis: The endpoint accepts a numeric order ID parameter without verifying whether the currently authenticated user session owns or has permission to view that specific transaction.

2. Exploitation

    Step 1: Logged into the application using a standard, low-privileged customer account (User B, assigned order order_id=8832).

    Step 2: Navigated to the order summary page and captured the request to GET /api/v1/orders/view?order_id=8832.

    Step 3: Modified the order_id parameter in the request from 8832 to 8831 (belonging to another customer) and forwarded the request to the server.

    Result: The application returned a 200 OK response containing the full name, shipping address, phone number, and purchased items of User A, successfully bypassing authorization boundaries.

3. Remediation

    Server-Side Access Control Checks: Implement robust authorization logic on the server side to cross-reference the logged-in user's session with the database owner ID of the requested order before returning data.

    Indirect Reference Maps: Replace direct database identifiers (like sequential integers) with unpredictable, cryptographically secure indirect references, such as UUIDs (e.g., order_id=3b1c9f42-a6e1-4c28-9d33-72ef419011ab).

    Principle of Least Privilege: Ensure API endpoints enforce strict tenant isolation and access control maps across all controller routes.

## Example 2: Healthcare Portal Patient Medical Record
1. Reconnaissance & Enumeration

    Discovery: During a security assessment of a telehealth web portal, web traffic was monitored while patients reviewed their electronic lab results.

    Endpoints Identified: The document viewer loads patient records via a predictable API call: GET /api/medical/records/download?record_id=10492.

    Vulnerability Hypothesis: The backend blindly trusts the sequential record_id integer provided by the client without verifying if the requesting patient session is authorized to access that record.

2. Exploitation

    Step 1: Logged into the healthcare portal using a test patient account (Patient B, record_id=10493).

    Step 2: Captured the outbound request for their own medical file in Burp Suite.

    Step 3: Decremented the record_id parameter value from 10493 to 10492 and sent the modified request.

    Result: The server responded with a 200 OK, returning confidential diagnostic reports, sensitive medical history, and prescription details belonging to Patient A, exposing sensitive Protected Health Information (PHI).

3. Remediation

    Server-Side Access Control Checks: Ensure every data fetch query maps the active session's user ID directly against the foreign key relationship in the medical records database.

    Indirect Reference Maps: Adopt cryptographically secure random identifiers (UUIDs) for sensitive healthcare records so they cannot be enumerated or guessed sequentially.

    Principle of Least Privilege: Implement granular authorization middleware to restrict clinical record access exclusively to verified clinical staff and the specific patient owner.

## Example 3: SaaS Enterprise Workspace & API Key Exposure
1. Reconnaissance & Enumeration

    Discovery: While auditing a multi-tenant SaaS project management tool, an analyst inspected API traffic during organization settings modification.

    Endpoints Identified: Organization configuration and API keys were fetched via GET /api/v1/orgs/settings?org_id=302.

    Vulnerability Hypothesis: The endpoint relies entirely on user authentication rather than tenant-level authorization, allowing any logged-in user to query arbitrary organization IDs.

2. Exploitation

    Step 1: Logged into the SaaS application under a legitimate, low-tier customer workspace (Organization A, org_id=303).

    Step 2: Captured the organization settings request and swapped the org_id parameter to 302 (belonging to a competing enterprise client).

    Step 3: Replayed the modified request to the server.

    Result: The application returned a 200 OK package containing Organization B's internal webhook secrets, active billing credit cards, third-party integration API keys, and internal user lists.

3. Remediation

    Server-Side Access Control Checks: Validate that the active user belongs to the requested organization tenant (org_id) before executing queries against configuration databases.

    Indirect Reference Maps: Use non-sequential, random hashes or UUIDs for workspace and organization identifiers across all API routes.

    Principle of Least Privilege: Enforce role-based access control (RBAC) to ensure that only designated tenant administrators can access sensitive organizational parameters.
