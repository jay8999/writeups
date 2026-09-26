# Vulnerability Report: CVE-2026-46300 (Fragnesia)

# 1. Reconnaissance & Enumeration

## CVE-2026-46300, codenamed Fragnesia, is a critical one-byte page-cache write primitive in the Linux kernel. It represents a regression introduced by an earlier security patch (the Dirty Frag fix) and highlights systemic audit gaps in kernel networking and subsystem code paths.

## Attack Surface & Environment Prerequisites

Shared Global Page Cache: The page cache is shared across all processes using the same kernel, including containers on the same host.

Containerized Environments: Rootless container deployments (such as default rootless Docker and Podman setups) allow unprivileged user namespaces. An attacker with namespace-root privileges has every prerequisite needed to execute Fragnesia. No privileged containers or special capabilities are required.

Target Subsystems: The vulnerability involves the interaction between memory management, socket buffers (sk_buff), skb_try_coalesce(), and the XFRM ESP-in-TCP receive path (esp_input()).

## Detection Challenges

File Integrity Tool Evasion: Because the attack corrupts pages directly in the kernel page cache without touching the on-disk file, traditional file integrity monitoring (FIM) and filesystem forensics will fail to surface the corruption after the fact.

Required Detection Layer: Detection must occur in real-time at the syscall layer by monitoring abnormal syscall sequences that legitimate applications do not produce.

# 2. Exploitation

The exploit chain bridges a 13-year-old latent bug with a security patch invariant violation, allowing controlled writes into arbitrary executable page-cache files (such as setuid binaries).

## The Five-Step Exploit Chain

Page Splicing: The attacker opens a TCP socket and splices target file pages (e.g., /usr/bin/su) from the page cache into the socket's receive queue, marking the socket buffers (skbs) with SKBFL_SHARED_FRAG.

Invariant Stripping: The kernel coalesces queued skbs via skb_try_coalesce(). During this process, the SKBFL_SHARED_FRAG marker is incorrectly stripped, making the skb appear as an ordinary private buffer to downstream functions.

Protocol Switching: The attacker configures the socket to ESP-in-TCP mode using setsockopt(TCP_ULP, "espintcp"), causing subsequent packet processing to treat queued data as ESP ciphertext.

In-Place Decryption Sink: esp_input() checks the (now-cleared) shared-frag flag, takes the fast path, and performs AES-GCM decryption directly in place over the page-cache page.

Controlled Byte Generation: Decryption applies a XOR of the AES-GCM keystream against the ciphertext. By iterating through the lower 32 bits of the 8-byte IV (up to 65,536 values), the attacker can deterministically select any target byte value to write into the page cache.

## Host Root Escalation (Two-Stage Process)

Stage 1 (Namespace Corruption): A pre-computed lookup table maps desired bytes to IV values. The exploit rapidly overwrites the first 192 bytes of /usr/bin/su with a custom ELF stub in a few hundred milliseconds.

Stage 2 (Host Root Execution): Because the page cache is globally shared, executing /usr/bin/su from outside the namespace forces the kernel to load the corrupted binary from the page cache. The setuid bit triggers an effective UID of 0, executing the payload to spawn a fully privileged host-root shell.

# 3. Remediation

## Mitigating Fragnesia requires immediate patching, temporary hardening controls, and changes to how kernel security patches are audited.

## Immediate Mitigations & Configuration Changes

Kernel Updates: Apply the vendor-provided kernel build that includes the merged Fragnesia fix as soon as it becomes available.

Module Denylisting: On hosts that do not rely on IPsec or AFS protocols, apply a modprobe denylist for vulnerable modules (esp4, esp6, and rxrpc). This serves as an effective interim control against both the primary exploit and unpatched secondary variants (such as the skb_segment() variant).

Production Dependencies: Environments running products like strongSwan, libreswan, or AFS clients must wait for vendor-backported patches if module denylisting cannot be applied.

Secure Coding & Patching Best Practices

Treat Patches as Attack Surface Changes: Kernel patches in complex subsystems (such as XFRM, AF_ALG, io_uring, BPF, and RxRPC) should be treated as alterations to the attack surface rather than completed mitigations. Follow-up audits of interacting functions are mandatory.

Invariant Auditing: Ensure that any function transferring fragments between buffers (such as skb_try_coalesce() and pskb_copy()) strictly preserves security flags like SKBFL_SHARED_FRAG to prevent invariant decay.
