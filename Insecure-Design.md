# Vulnerability Writeup: Insecure Design (AS06)

Below is an example of how insecure design vulnerabilities are discovered, exploited, and remediated across traditional workflows and AI-enabled systems.

---

## 1. Reconnaissance & Enumeration

* **Discovery:** During architecture review and traffic interception with a proxy (such as Burp Suite), workflow assumptions and trust boundaries were mapped across client requests, backend APIs, and integrated AI features.
* **Endpoints Identified:** Core features relied on implicit client/model trust rather than server-side enforcement—for example, `POST /api/v1/assistant/query` executing model-generated database queries directly, or `POST /api/v1/account/recover` accepting unverified recovery parameters without secondary checks.
* **Vulnerability Hypothesis:** The system was built with flawed architectural assumptions—trusting client-side flows, unvalidated model outputs, or weak business logic controls due to skipped threat modelling and missing abuse-case reviews.

---

## 2. Exploitation

* **Step 1:** Authenticated as a standard, low-privileged user (or interacted with an unauthenticated/lightly guarded workflow) and observed how inputs pass through business logic or AI components.
* **Step 2:** Captured requests in Burp Suite and identified where untrusted input shapes execution—such as blending user-supplied text into an AI prompt/tool call, bypassing an approval/recovery step, or invoking backend endpoints directly outside the intended UI flow.
* **Step 3:** Sent crafted payloads designed to abuse the workflow or hijack model/service behavior (e.g., instructing an AI agent with unchecked tool access to query unauthorized records, or skipping a step in a multi-stage business process).
* **Result:** The application executed unauthorized actions or returned restricted data because the flaw was embedded in the workflow logic and trust boundaries rather than a simple missing input check.

---

## 3. Remediation

* **Threat Modelling & Secure-by-Design Requirements:** Conduct threat modelling across every development stage—including AI-specific risks such as prompt injection, agent misuse, inference exposure, and poisoned models—and define explicit security requirements before coding.
* **Zero Trust for AI & Model Outputs:** Treat every model as untrusted. Separate system instructions from user content, validate and filter all inputs and outputs, keep sensitive data out of prompts unless strictly necessary, and enforce human-in-the-loop approval for high-risk actions.
* **Strict Business Logic, Least Privilege & Guardrails:** Enforce server-side authorization, session management, and least privilege across users, APIs, and AI tools/agents; remove debug or test bypasses before production; verify third-party/model supply chains; and continuously monitor for logic abuse.

---

## Examples

### Example 1: AI SQL Assistant with Unchecked Query Execution

**Reconnaissance & Enumeration**

* **Discovery:** While assessing an internal analytics web app featuring a natural-language AI assistant, requests to the reporting feature were inspected in Burp Suite.
* **Endpoints Identified:** Users asked questions via `POST /api/v1/ai/report`, where the backend passed the prompt to an LLM and ran the resulting SQL query against the reporting database.
* **Vulnerability Hypothesis:** The architecture assumes the LLM only generates safe, scoped `SELECT` statements for the current user's tenant, lacking query sandboxing, least-privilege DB credentials, or output validation.

**Exploitation**

* **Step 1:** Logged in with a low-privileged analyst account restricted to department-level sales summaries.
* **Step 2:** Captured the `POST /api/v1/ai/report` request and submitted a prompt crafted via prompt injection to override system instructions and query salary/HR tables across all departments.
* **Step 3:** Forwarded the request and observed the backend execute the model-generated query directly.
* **Result:** The application returned sensitive compensation and employee records outside the analyst's authorized scope, exposing a design flaw where unvalidated model output drove privileged data access.

**Remediation**

* **Threat Modelling & Secure-by-Design Requirements:** Model prompt-injection and data-exfiltration paths during feature design and enforce explicit data-access boundaries before deploying AI query features.
* **Zero Trust for AI & Model Outputs:** Separate system instructions from user input, parse and allowlist generated queries/actions before execution, and require human review for high-impact or broad data operations.
* **Strict Business Logic, Least Privilege & Guardrails:** Run AI-backed queries using a read-only, tenant-scoped database role with strict row/column permissions, log model provenance and behavior, and continuously test for logic bypasses.

---

### Example 2: Flawed Account Recovery & Approval Workflow

**Reconnaissance & Enumeration**

* **Discovery:** During a security review of a customer portal, password reset and account recovery traffic was captured using an interception proxy.
* **Endpoints Identified:** Recovery progressed through `POST /api/v1/auth/recovery/init`, `POST /api/v1/auth/recovery/verify`, and `POST /api/v1/auth/recovery/complete`.
* **Vulnerability Hypothesis:** The recovery state machine assumes users always follow the UI sequence in order and relies on a client-controllable or weakly bound verification flag rather than enforcing server-side step and token validation.

**Exploitation**

* **Step 1:** Initiated account recovery for a test account in Burp Suite and mapped the sequence of API calls.
* **Step 2:** Skipped or tampered with the verification step/state transition and sent `POST /api/v1/auth/recovery/complete` directly with a new password payload.
* **Step 3:** Replayed the request to complete the state change.
* **Result:** The server accepted the password reset without completing proper verification, demonstrating weak business logic controls built into the recovery workflow.

**Remediation**

* **Threat Modelling & Secure-by-Design Requirements:** Map abuse cases for multi-step workflows (recovery, checkout, approvals) during design and enforce strict server-side state machines.
* **Zero Trust for AI & Model Outputs:** If AI or automated agents participate in verification or support approvals, treat their decisions as untrusted and require deterministic server checks and human review for sensitive changes.
* **Strict Business Logic, Least Privilege & Guardrails:** Bind short-lived, single-use verification tokens strictly to the user, session, and step on the server; enforce authentication/authorization at every transition; and test workflow abuse paths regularly.

---

### Example 3: Direct API Bypass of Mobile-Only Trust Assumptions

**Reconnaissance & Enumeration**

* **Discovery:** While auditing a social audio/community platform (similar to the Clubhouse architecture pattern), API traffic was intercepted outside the official mobile client.
* **Endpoints Identified:** Room metadata, participant lists, and live stream/room endpoints were exposed via predictable REST/RPC calls such as `GET /api/v1/rooms/{room_id}` and `GET /api/v1/rooms/{room_id}/participants`.
* **Vulnerability Hypothesis:** The system was designed under the flawed assumption that users would only interact through the first-party mobile app UI, leaving backend endpoints without proper authentication and authorization checks.

**Exploitation**

* **Step 1:** Captured API requests generated by the mobile client and extracted room and user identifiers.
* **Step 2:** Replayed direct requests to backend endpoints using an HTTP client/proxy outside the mobile app flow, including targeting rooms marked as private.
* **Step 3:** Enumerated room and user endpoints directly against the API.
* **Result:** The backend returned room details, user profiles, and private room/conversation metadata without enforcing proper access controls, breaking the platform's privacy guarantees due to a foundational design assumption.

**Remediation**

* **Threat Modelling & Secure-by-Design Requirements:** Never treat client UI restrictions or mobile-only usage assumptions as a security boundary; perform threat modelling across direct API access and automated clients.
* **Zero Trust for AI & Model Outputs:** Do not let automated assistants or external integrations bypass authorization or expose internal metadata/prompts through unauthenticated or overly broad endpoints.
* **Strict Business Logic, Least Privilege & Guardrails:** Enforce strong authentication, authorization, and session/token validation on every backend endpoint; apply least privilege to data responses; and continuously monitor API traffic for anomalous access patterns.
