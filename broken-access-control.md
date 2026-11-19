# OWASP Top 10 — A01: Broken Access Control

**Broken Access Control** occurs when an application fails to properly verify whether an authenticated or unauthenticated user is authorized to perform a requested action or view a requested resource on every request. Because access control decisions are often missing on backend routes, APIs, or object queries—or rely on client-supplied parameters, roles, or identifiers—attackers can bypass intended boundaries.

In practice, this manifests as **Insecure Direct Object Reference (IDOR)** (such as changing a parameter like `?id=7` to `?id=6` to access another record), **horizontal privilege escalation** (accessing another user's resources at the same privilege level), or **vertical privilege escalation** (performing administrative or privileged actions from a standard or unauthenticated account).

---

## 1. Reconnaissance & Enumeration

During the initial assessment of the web application, the goal was to identify endpoints, object identifiers, and role/permission boundaries susceptible to broken access control.

* **Mapping Application Routes & Role Boundaries:** Cataloged user-facing pages, REST/GraphQL endpoints, and administrative or internal routes across different privilege levels (anonymous, standard user, moderator, admin) to identify missing authorization checks or exposed privileged functionality.
* **Inspecting Direct Object References & Parameters:** Analyzed query strings, path parameters, request bodies, and headers (such as user IDs, invoice IDs, document UUIDs, or tenant identifiers) to see how backend resources were addressed and whether ownership was validated server-side.
* **Analyzing Client-Side State & Role Indicators:** Examined hidden form fields, local/session storage, JWT claims, cookies, and frontend UI gating to see if roles, permissions, or account states were asserted on the client rather than enforced authoritatively on the server.
* **Testing Cross-User & Cross-Role Requests:** Validated access controls by replaying requests from one session or role using another session's token or identifier, observing whether the server returned authorized data, HTTP `200`/`201`/`204` responses, or proper `401`/`403` denials.

---

## 2. Exploitation

Once gaps in server-side authorization or object-level ownership checks were identified, weaknesses were leveraged to access restricted data or execute unauthorized actions:

* **Manipulating Object Identifiers (IDOR):** Modified sequential or predictable resource IDs and parameters in requests (for example, changing a profile, order, or file identifier to another user's value) to read, update, or delete records belonging to other accounts.
* **Performing Horizontal Privilege Escalation:** Replayed authenticated requests belonging to one standard user against resources owned by another user of the same role, exposing private data, messages, settings, or account actions.
* **Performing Vertical Privilege Escalation:** Accessed administrative or privileged endpoints directly (forced browsing), invoked restricted HTTP methods (`PUT`, `PATCH`, `DELETE`), or altered role/privilege fields in requests or tokens to gain elevated capabilities.
* **Gaining Unauthorized Data & Functional Access:** Extracted sensitive user records, PII, internal configuration data, or performed state-changing actions without proper permission enforcement.

---

## 3. Remediation

To effectively remediate broken access control and enforce least privilege across the application, developers should implement the following defensive practices:

* **Enforce Deny-by-Default Access Control Server-Side:** Centralize authorization logic in reusable middleware or policy layers that deny access by default and verify permissions on every single request—never relying solely on frontend UI hiding or client-supplied role flags.
* **Validate Object-Level Ownership (Prevent IDOR):** Check on every object read, update, or delete that the authenticated user (or tenant) is explicitly authorized to access that specific resource ID, and use hard-to-guess identifiers (such as UUIDs) alongside—not as a replacement for—authorization checks.
* **Enforce Role-Based or Attribute-Based Checks on All Routes & Methods:** Explicitly verify user roles, permissions, and account status across all endpoints, APIs, and HTTP verbs, including administrative functions, file downloads, and bulk operations.
* **Log Access Control Failures & Alert on Anomalies:** Record failed authorization attempts and repeated probing of object IDs or privileged routes so security teams can detect enumeration or escalation attempts early.

---

## Examples

### Example 1: Insecure Direct Object Reference (IDOR) on User Invoices

* **Reconnaissance & Enumeration:** While reviewing billing functionality, a standard user requests `/api/v1/invoices?id=1042` and receives their own invoice JSON. Changing the parameter to `/api/v1/invoices?id=1041` returns another customer's billing details with a `200 OK` response, showing the backend fetches records by `id` without checking whether `invoice.user_id` matches the authenticated user.
* **Exploitation:** Using a script or proxy repeater, the attacker iterates across sequential invoice IDs (`1000` to `5000`), harvesting customer names, addresses, and payment summaries across accounts without authorization.
* **Remediation:** Enforce a server-side ownership check tying every query and mutation to the authenticated user's ID (and tenant ID where applicable), returning `403 Forbidden` (or `404 Not Found` consistently) when a user requests a resource they do not own.

### Example 2: Vertical Privilege Escalation via Missing Admin Role Check

* **Reconnaissance & Enumeration:** During endpoint mapping, a standard user discovers `/api/v1/admin/users` and `/api/v1/admin/users/45/role` in API responses or frontend bundles. Sending a `GET` request to `/api/v1/admin/users` with a standard user's session token succeeds instead of returning `403 Forbidden`.
* **Exploitation:** The attacker sends a `PATCH` request to `/api/v1/admin/users/45/role` with `{"role": "admin"}` using their own session token. Because the endpoint only checks for an authenticated session rather than an `admin` role, the attacker elevates their own account to administrator.
* **Remediation:** Apply strict role/permission authorization middleware on all `/api/v1/admin/*` routes and individual actions, ensuring privilege changes require verified administrative authorization and re-evaluation of active permissions.

### Example 3: Horizontal Privilege Escalation by Tampering with `user_id` in Profile Updates

* **Reconnaissance & Enumeration:** When updating profile details via `PUT /api/v1/profile`, the request body includes `"user_id": 88` alongside `"display_name"`. Testing reveals the backend updates the record matching the `user_id` field in the JSON payload rather than deriving the target user exclusively from the verified server-side session.
* **Exploitation:** The attacker changes `"user_id"` from `88` to `89` in the `PUT /api/v1/profile` request body and submits it with their own authentication token, overwriting another user's profile information and contact details.
* **Remediation:** Derive the acting user's identity strictly from the verified server-side session or token context, ignore or reject client-supplied owner IDs on self-service endpoints, and validate explicit authorization for any administrative cross-user edits.
