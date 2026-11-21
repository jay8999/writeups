# Vulnerability Writeup: Software Supply Chain Failures (AS03)

Below is an overview of how software and AI supply chain weaknesses are identified, leveraged conceptually, and remediated.

## 1. Reconnaissance & Enumeration

During the initial assessment of the application and deployment environment, the focus is on mapping external dependencies, build pipelines, and model sources that could introduce untrusted behavior:

* **Identifying High-Risk Dependency & Model Surfaces:** Mapped third-party packages, container base images, external APIs, fine-tuned datasets, and pre-trained AI models integrated into critical workflows.
* **Analyzing Verification & Update Practices:** Checked whether packages, updates, or weights are pulled automatically without cryptographic signature verification, checksum validation, or version pinning.
* **Inspecting Build & Provenance Controls:** Reviewed CI/CD workflows, artifact registries, and SBOM/provenance records to see if build steps or external sources lack integrity enforcement and runtime oversight.
* **External & Supply-Path Testing:** Evaluated dependency resolution paths and staging environments to verify whether unverified artifacts or anomalous package behaviors can enter the build or execution flow without being blocked.

## 2. Exploitation

When a supply chain link lacks integrity checks, provenance validation, or isolation, adversaries can introduce unauthorized logic or hidden behavior upstream:

* **Crafting the Compromised Artifact:** An adversary introduces unauthorized code, tampered build steps, or hidden/backdoored behavior into a package, update mechanism, model weight, or training/fine-tuning dataset rather than modifying the target application's core codebase directly.
* **Leveraging Trust & Automation:** Because downstream systems often trust upstream registries, signed-off pipelines, or popular third-party models by default, automated builds or updates can pull the altered artifact without manual review.
* **Triggering Execution:** When the application compiles, deploys, or runs inference using the compromised component or model, the embedded behavior activates within the trusted context of the host environment.
* **Gaining Unauthorized Impact:** The attacker achieves unauthorized access, data exposure, or manipulated model outputs across downstream environments that rely on the affected link.

## 3. Remediation

To reduce supply chain risk across code, infrastructure, and AI components, teams should enforce verification and containment throughout the lifecycle:

* **Verify Components & Models Before Use:** Vet third-party libraries, containers, datasets, and AI models against trusted sources, integrity hashes, and security baselines prior to adoption.
* **Sign & Audit Updates and Artifacts:** Require cryptographic signing and verification for releases, packages, and model artifacts, while maintaining tamper-evident build logs and provenance tracking.
* **Lock Down CI/CD & Dependency Hygiene:** Restrict pipeline permissions, pin versions where appropriate, and continuously monitor deployed dependencies for known vulnerabilities and unmaintained packages.
* **Monitor Runtime & Model Behavior:** Observe runtime activity and AI model outputs for anomalous behavior, unexpected network or file access, or data leakage, backed by supply chain threat modeling across the SDLC.

---

## Examples

### Example 1: Compromised Build & Update Pipeline

1. **Reconnaissance & Enumeration**
2. **Identifying High-Risk Surfaces:** Focused on the release build pipeline and automated update distribution channel feeding production services.
3. **Analyzing Verification Practices:** Found that downstream deployments automatically applied updates without strict end-to-end build provenance and integrity checks across every stage.
4. **External Testing:** Assessed pipeline boundaries and staging rollouts to confirm that unverified build modifications could propagate if build integrity were bypassed.
5. **Exploitation**
6. **Crafting the Compromised Artifact:** Malicious logic is inserted during the build or update packaging stage rather than inside the application's primary source repository.
7. **Leveraging Trust & Automation:** Automated update mechanisms treat the resulting release package as legitimate and distribute it broadly.
8. **Triggering Execution & Gaining Impact:** Organizations installing the update execute the embedded logic inside their trusted infrastructure, enabling broad downstream compromise.
9. **Remediation**
10. **Lock Down CI/CD & Sign Releases:** Harden build environments, enforce least privilege, and require cryptographic signature and provenance verification before deployment.
11. **Monitor Runtime Behavior:** Track anomalous process, file, or network activity in production to detect unexpected changes introduced via updates.

### Example 2: Unpinned or Unverified Third-Party Dependency

1. **Reconnaissance & Enumeration**
2. **Identifying High-Risk Surfaces:** Examined application manifests and package installation steps pulling external open-source libraries.
3. **Analyzing Verification Practices:** Observed floating version ranges, unvetted transitive dependencies, and lack of continuous post-deployment vulnerability monitoring.
4. **External Testing:** Checked dependency resolution and staging builds to verify how newly published or altered packages are ingested.
5. **Exploitation**
6. **Crafting the Compromised Artifact:** An upstream package version or dependency link is altered to introduce malicious or data-exfiltrating functionality.
7. **Leveraging Trust & Automation:** Automated dependency resolution or rebuilds pull the altered package into the application automatically.
8. **Triggering Execution & Gaining Impact:** The compromised library runs within the application process, exposing sensitive data or granting unauthorized capabilities.
9. **Remediation**
10. **Verify & Track Provenance:** Audit dependencies, maintain SBOMs/provenance records, and enforce strict artifact vetting before promotion.
11. **Monitor & Patch Continuously:** Continuously scan dependencies after deployment and isolate or patch affected packages promptly.

### Example 3: Untrusted Third-Party AI Model or Fine-Tuned Dataset

1. **Reconnaissance & Enumeration**
2. **Identifying High-Risk Surfaces:** Investigated an AI feature relying on an external pre-trained model and community fine-tuning data.
3. **Inspecting Verification & Controls:** Found that model weights and dataset sources were adopted without integrity/provenance checks, red-teaming, or output/runtime monitoring.
4. **External Testing:** Evaluated model behavior in a controlled test environment to see how inputs triggered unexpected or unsafe responses.
5. **Exploitation**
6. **Crafting the Compromised Artifact:** Hidden triggers, backdoors, or biased behaviors are embedded within the third-party model weights or training/fine-tuning data.
7. **Leveraging Trust & Automation:** The application loads the external model directly into production inference pipelines without thorough validation.
8. **Triggering Execution & Gaining Impact:** Specific inputs or operational conditions trigger the embedded behavior, leading to bypassed guardrails, biased outputs, or sensitive data leakage.
9. **Remediation**
10. **Verify & Audit AI Components:** Validate model sources, provenance, and data hygiene, and test models before integrating them into workflows.
11. **Enforce Runtime Guardrails & Monitoring:** Apply input/output controls, sandboxing, and continuous monitoring for anomalous model behavior and data exposure.
