# Vulnerability Writeup: Software or Data Integrity Failures (OWASP A08)

Below is an overview of how Software or Data Integrity Failures occur conceptually and how to remediate them.

## 1. Reconnaissance & Enumeration

During the initial assessment of the application and delivery pipeline, the goal was to identify trust boundaries and components lacking integrity or authenticity checks.

* **Identifying Untrusted Inputs & Dependencies:** Focused on areas where the application loads external libraries, plugins, configuration files, or serialized data without verifying source authenticity.
* **Analyzing Update & Build Mechanisms:** Checked how software updates, patches, and build artifacts are fetched, signed, and deployed across environments.
* **Inspecting Integrity Controls:** Examined whether code, packages, or sensitive payloads used cryptographic verification (such as digital signatures or checksums) or schema/integrity validation prior to processing.
* **Pipeline & Boundary Review:** Evaluated access controls and separation of duties across source control, build runners, and deployment stages to spot unverified handoffs.

## 2. Exploitation

When an application or pipeline trusts unverified code, updates, or data, integrity can be subverted at a conceptual level:

* **Manipulating Unverified Artifacts or Data:** An adversary introduces altered code, configuration data, or serialized payloads into a path that the target system accepts without integrity verification.
* **Bypassing Trust Assumptions:** Because the consumer assumes incoming artifacts or data originate from a trusted source in an unaltered state, verification checks are skipped or absent.
* **Triggering Execution or Logic Alteration:** When the application processes the unverified update, script, or data structure, unintended behavior or state changes occur within the target environment.
* **Impact Realization:** Unauthorized logic execution, tampered application state, or compromised build outputs can occur depending on where the missing integrity check resides.

## 3. Remediation

To reduce Software or Data Integrity Failures, teams should enforce cryptographic verification and strict trust boundaries across code, data, and delivery pipelines:

* **Verify Signatures and Checksums:** Require cryptographic signatures or trusted hashes for software updates, dependencies, and packages before installation or execution.
* **Enforce Strict Data & Deserialization Controls:** Validate schemas and integrity on external data, templates, and serialized objects, avoiding insecure deserialization of untrusted inputs.
* **Secure CI/CD Pipelines:** Restrict access to repositories and build systems, isolate build environments, and verify provenance of build artifacts and dependencies.
* **Pin Trusted Sources:** Reference vetted registries and controlled repositories with pinned versions or digests rather than mutable or unverified external sources.

---

## Examples

### Example 1: Unverified Application Update Mechanism

1. Reconnaissance & Enumeration
2. Identifying Untrusted Inputs & Dependencies: Inspected the client updater routine that downloads release packages over the network.
3. Analyzing Update & Build Mechanisms: Found that downloaded update bundles were applied automatically without verifying a digital signature or expected checksum.
4. Inspecting Integrity Controls: Confirmed no cryptographic key check prevented modified or corrupted packages from being staged.
5. Pipeline & Boundary Review: Reviewed release distribution steps to see how packages reached the update endpoint.
6. Exploitation
7. Manipulating Unverified Artifacts or Data: In a controlled test scenario, an unverified update package was altered to represent an unauthorized release artifact.
8. Bypassing Trust Assumptions: The updater trusted the delivery path and applied the package without signature validation.
9. Triggering Execution or Logic Alteration: The application installed the altered update files during its routine update cycle.
10. Impact Realization: Unverified code was introduced into the application runtime, demonstrating risks to software integrity.
11. Remediation
12. Verify Signatures and Checksums: Sign all release packages with a trusted key and verify signatures and hashes before applying updates.
13. Enforce Strict Data & Deserialization Controls: Reject unsigned, malformed, or incomplete update manifests prior to extraction.

### Example 2: Untrusted External Script or Plugin Loading

1. Reconnaissance & Enumeration
2. Identifying Untrusted Inputs & Dependencies: Examined modules that dynamically load external scripts, themes, or plugins into the application.
3. Analyzing Update & Build Mechanisms: Checked how remote or third-party assets were fetched and integrated into runtime behavior.
4. Inspecting Integrity Controls: Observed that external resources lacked integrity checks or strict origin verification before execution.
5. Exploitation
6. Manipulating Unverified Artifacts or Data: Evaluated how a modified external script or plugin asset would be handled if served from an untrusted or altered source.
7. Bypassing Trust Assumptions: The application treated external code assets as inherently safe once retrieved.
8. Triggering Execution or Logic Alteration: Loaded logic executed directly within the application context without integrity validation.
9. Impact Realization: Unauthorized behavior could execute within user or application sessions due to unverified external code.
10. Remediation
11. Pin Trusted Sources: Host critical assets internally or use vetted sources with subresource integrity and pinned versions.
12. Verify Signatures and Checksums: Require integrity verification for plugins, extensions, and executable modules prior to loading.

### Example 3: Unprotected Build Pipeline & Artifact Handoff

1. Reconnaissance & Enumeration
2. Identifying Untrusted Inputs & Dependencies: Audited build scripts and automated deployment workflows handling source code and dependencies.
3. Analyzing Update & Build Mechanisms: Identified overly broad write permissions and weak verification between build steps and deployment targets.
4. Inspecting Integrity Controls: Found that build artifacts lacked provenance attestation or integrity checks when promoted between stages.
5. Exploitation
6. Manipulating Unverified Artifacts or Data: Assessed how tampering with build configuration or intermediate artifacts could affect downstream deployments.
7. Bypassing Trust Assumptions: Deployment stages assumed artifacts produced by earlier steps were authentic and unaltered.
8. Triggering Execution or Logic Alteration: Altered build instructions or artifacts propagated into staging or production workflows.
9. Impact Realization: Compromised integrity across the delivery pipeline could affect deployed application code or configuration.
10. Remediation
11. Secure CI/CD Pipelines: Enforce least privilege, code review, protected branches, and isolated build runners across CI/CD stages.
12. Verify Signatures and Checksums: Generate signed build provenance and verify artifact integrity before production deployment.
