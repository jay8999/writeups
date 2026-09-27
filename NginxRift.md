# Vulnerability Report: NGINX Rift (CVE-2026-42945)

## Severity: Critical (CVSS v4 Base Score: 9.2)

## Affected Software: NGINX Open Source 0.6.27 through 1.30.0, NGINX Plus R32 through R36, and downstream F5 products (NGINX App Protect, Gateway Fabric, Ingress Controller).

The published sequence proceeds in four stages. An attacker first opens a connection and sends only partial HTTP headers, which causes NGINX to allocate a request pool but not to start processing the request.

A second connection is opened immediately after, causing the pool allocator to place the second connection's pool directly adjacent to the first. The attacker then completes the headers of the first connection in a way that triggers the rewrite overflow, so the overflow runs out of the first pool and into the header of the second, adjacent pool.

Finally, the attacker closes the second connection, which causes NGINX to call ngx_destroy_pool on the corrupted pool. 

The destroy path walks the cleanup list and never touches the other corrupted fields, so the worker calls the attacker's chosen handler with the attacker's chosen argument and does not crash before reaching it.

# 1. Reconnaissance & Enumeration
## 1.1 Target Identification & Service Discovery

  Service: NGINX Web Server / Reverse Proxy.

  Architecture: Master-worker process model where a privileged master process spawns identical unprivileged worker processes sharing an identical memory address layout.

  Port mapping: Target service typically runs on standard HTTP/HTTPS ports or mapped container ports (e.g., 19321/tcp).

  Verification: Benign requests to a vulnerable location block return standard responses (e.g., HTTP/1.1 200 OK), while service availability and versioning can be fingerprinted via standard banner grabbing or package queries (nginx -v).

## 1.2 Vulnerability Profiling

CVE-2026-42945 ("NGINX Rift") is a heap buffer overflow in the ngx_http_rewrite_module. It is unauthenticated and deterministic in memory layouts across worker processes.

Trigger Condition: The bug is not reachable on every installation. It strictly requires a configuration pattern combining:

  A rewrite directive containing an unnamed PCRE capture (e.g., $1, $2).

  A replacement string containing a question mark (?).

  A subsequent rewrite, if, or set directive in the same location block scope referencing that capture.

Vulnerable Configuration Example:

      location ~ ^/api/(.*)$ {
          rewrite ^/api/(.*)$ /internal?migrated=true;
          set $original_endpoint $1;
      }

# 2. Exploitation

## 2.1 Root Cause Analysis

The flaw stems from an internal state mismatch regarding the is_args flag on struct ngx_http_script_engine_t:

  When a rewrite replacement string includes a question mark, the compiled bytecode sets e->is_args = 1 on the main script engine to denote query-string processing. This flag is never reset for subsequent directives in the same block.

  When a subsequent set directive evaluates a regex capture using ngx_http_script_complex_value_code, it runs a length-calculation pass against a freshly zeroed sub-engine (le.is_args = 0).

  However, the subsequent copy/write pass runs on the main engine (e->is_args = 1).

  The Mismatch: For URI-escapable characters (like the plus sign +), the length pass counts the character as 1 byte (unescaped), whereas the copy pass escapes it into a 3-percent-encoded form (%2B), writing 3 bytes into a buffer allocated for only 1 byte. This yields a deterministic, attacker-controlled heap overflow on the worker's per-request memory pool (ngx_pool_t).

## 2.2 Execution Mechanics (Heap Feng Shui & RCE)

Because direct arbitrary pointer writes are restricted by URI-safe encoding limits, exploitation leverages a cross-request heap layout technique:

  Adjacent Pool Allocation: An attacker sends partial headers on Connection A (allocating a request pool) followed immediately by Connection B (allocating an adjacent pool).

  Overflow Trigger: Completing Connection A triggers the rewrite overflow, spilling raw data from Pool A into the header structure of Pool B.

  Cleanup Trigger: Closing Connection B forces NGINX to call ngx_destroy_pool on the corrupted pool. This walks the cleanup callback list (ngx_pool_cleanup_t), triggering a function pointer call (handler(data)).

  Brute-Forcing via Sprays: By utilizing HTTP POST bodies (which bypass URI escaping and support null bytes) to spray fake cleanup structures and leveraging the master process's automatic worker respawn behavior on crashes, the exploit iteratively converges on a successful code execution vector (e.g., via system()).

## 2.3 Step-by-Step Exploitation Workflow

Confirm Container State:

    sudo docker ps --filter label=com.docker.compose.project=nginx-rift

Run Single-Command Mode (PoC):
Execute the exploit driver to verify execution by creating a marker file:

    python3 poc.py --cmd 'echo hello from depthfirst > /tmp/pwned'

Run Reverse Shell Mode:
Spawn a local listener and trigger an interactive worker shell:

    python3 poc.py --shell

# 3. Remediation
## 3.1 Official Patches (Primary Fix)

Upgrade NGINX to patched versions immediately:

  NGINX Open Source: Upgrade to 1.30.1 or 1.31.0.

  NGINX Plus: Upgrade to R32 P6, R36 P4, or 37.0.0 (LTS track).

  Downstream Products: Apply vendor-supplied patches for F5, NGINX App Protect WAF/DoS, Gateway Fabric, and Ingress Controller.

## 3.2 Configuration Workaround (Temporary Mitigation)

If an immediate binary upgrade is not feasible, modify the configuration to use named captures instead of unnamed captures. Named captures invoke a distinct evaluation engine path that is unaffected by the is_args state mismatch:

    # Mitigated Configuration using Named Captures
    location ~ ^/api/(?<path>.*)$ {
        rewrite ^/api/(?<path>.*)$ /internal?migrated=true;
        set $original_endpoint $path;
    }

## 3.3 Detection & Monitoring

Access Logs: Inspect access logs for anomalous bursts of unusually long request URIs containing high concentrations of escapable characters (e.g., long sequences of + symbols):

    grep -E '\+{30,}' /var/log/nginx/access.log

Error Logs: Monitor error logs for repeated worker crashes (N exited on signal 11), which indicate potential exploitation or denial-of-service attempts.







  
