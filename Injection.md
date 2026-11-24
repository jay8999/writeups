# Vulnerability Writeup: Injection

Below is an example of how this vulnerability is often exploited and a proposal of a solution.

## 1. Reconnaissance & Enumeration

During the initial assessment of the application, the goal was to identify inputs and interpreters susceptible to Injection flaws where untrusted data alters command or query execution.

* **Identifying Input & Interpreter Boundaries:** Focused on parameters, headers, or fields passed into downstream interpreters such as SQL databases, operating system shells, template engines, or LLM prompts.
* **Analyzing Input Handling:** Checked whether user-supplied values were concatenated or interpolated directly into query strings, commands, or templates without parameterization or context-aware escaping.
* **Inspecting Validation & Escaping Controls:** Examined whether input validation, strict type enforcement, or safe APIs were applied before data reached the execution layer.
* **Safe Boundary Testing:** Validated findings conceptually by observing how benign or boundary inputs affected query structure or execution behavior without deploying harmful payloads.

## 2. Exploitation

Once an insecure input path lacking proper separation between data and executable instructions was identified, the flaw was leveraged to alter intended interpreter logic:

* **Crafting the Input Concept:** Designed test input intended to break out of the data context and influence command, query, or template structure rather than being treated purely as literal data.
* **Leveraging Direct Interpolation:** Because the application built commands or queries dynamically from raw input, the interpreter parsed untrusted segments as structural syntax or directives.
* **Triggering Alternate Execution:** When the crafted input reached the backend interpreter, the underlying engine executed unintended logic or altered query conditions.
* **Gaining Unauthorized Access or Impact:** The application returned unauthorized data, modified state, or executed unintended operations according to the manipulated interpreter instructions.

## 3. Remediation

To effectively prevent Injection vulnerabilities and keep untrusted data separate from executable instructions, developers should implement the following defensive practices:

* **Use Parameterized Queries & Safe APIs:** Use prepared statements and parameterized queries for database access, and avoid invoking OS shells with raw user data by using safe APIs or direct argument arrays.
* **Enforce Context-Aware Escaping & Sandboxing:** Apply strict output encoding or template sandboxing when rendering dynamic content, and separate system instructions from untrusted user content in AI prompt workflows.
* **Validate & Normalize Inputs:** Enforce allow-lists, strict data types, and length/format constraints on incoming data as a defense-in-depth layer before processing.
* **Apply Least Privilege:** Restrict database accounts, service credentials, and runtime permissions to only the resources and operations strictly required.

---

## Examples

### Example 1: Database Query Manipulation via Unparameterized Input (SQL Injection Concept)

1. Reconnaissance & Enumeration
2. Identifying Input & Interpreter Boundaries: Targeted a search or authentication parameter passed into a relational database query.
3. Analyzing Input Handling: Discovered the backend constructed SQL queries by concatenating user input directly into query strings rather than using parameterized statements.
4. Inspecting Validation & Escaping Controls: Observed that input reached the query builder without bound parameters or strict type enforcement.
5. Exploitation
6. Crafting the Input Concept: Prepared input containing syntax intended to alter the logical structure of the query condition.
7. Leveraging Direct Interpolation: Because the query string was assembled dynamically, the database parser evaluated the injected syntax as part of the SQL statement.
8. Triggering Alternate Execution & Gaining Impact: The modified query altered record selection or access logic, exposing or affecting data beyond what the application intended.
9. Remediation
10. Use parameterized queries and prepared statements so user input is always bound strictly as data values.
11. Restrict database user privileges to least privilege and enforce strict input validation.

### Example 2: Unintended Shell Execution via Dynamic Command Assembly (Command Injection Concept)

1. Reconnaissance & Enumeration
2. Identifying Input & Interpreter Boundaries: Focused on an administrative or utility feature that passes a parameter to an underlying system command.
3. Analyzing Input Handling: Found that the application passed user-supplied values into a shell invocation instead of using a direct, non-shell execution API.
4. Inspecting Validation & Escaping Controls: Identified missing allow-list validation and lack of strict separation between command arguments and control characters.
5. Exploitation
6. Crafting the Input Concept: Constructed input designed to introduce command-separator syntax within a parameter passed to the shell.
7. Leveraging Direct Interpolation: Because a system shell interpreted the combined command string, control characters altered command execution boundaries.
8. Triggering Alternate Execution & Gaining Impact: The host environment ran unintended commands under the application process identity, risking unauthorized file or system access.
9. Remediation
10. Replace shell invocations with safe APIs or argument-list execution that does not invoke a command shell.
11. Validate inputs against strict allow-lists and run services under least-privileged accounts.

### Example 3: Template Logic Alteration via Unsanitized Expressions (Server-Side Template Injection Concept)

1. Reconnaissance & Enumeration
2. Identifying Input & Interpreter Boundaries: Investigated a customizable display or message field rendered through a server-side template engine.
3. Analyzing Input Handling: Reconnaissance showed user input was embedded directly inside template evaluation context rather than passed safely as a plain display variable.
4. Inspecting Validation & Escaping Controls: Found weak or missing template sandboxing and insufficient filtering of template expression delimiters.
5. Exploitation
6. Crafting the Input Concept: Formulated input structured around template syntax delimiters to test whether expressions were evaluated server-side.
7. Leveraging Direct Interpolation: The template engine parsed the embedded expression syntax instead of treating the entire string as static text.
8. Triggering Alternate Execution & Gaining Impact: Server-side expression evaluation exposed internal context or altered rendered output beyond standard text substitution.
9. Remediation
10. Pass user values strictly as data context into logic-less or sandboxed templates, avoiding direct evaluation of untrusted template expressions.
11. Enforce strict input validation and keep template engines updated with security hardening enabled.
