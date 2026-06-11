---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: SSH
description: "Connects to the advertised SSH ports of a server and audits reachability, banner-to-CVE matches, the full algorithm posture, observed host keys, SSHFP alignment and authentication methods."
weight: 280
---

The **SSH** checker produces a comprehensive security audit of the SSH service exposed by a *Server*. It connects to the advertised SSH port(s) on every `A`/`AAAA` address and reports reachability, banner-to-CVE matches, the full algorithm posture (key exchange, host-key, cipher, MAC, compression), the observed host keys, SSHFP fingerprint alignment, and the authentication methods the server exposes. Results are presented as a "fix me fast" HTML report.

**Scope:** service-level. It attaches to services of type `abstract.Server` (a subdomain that publishes `A`/`AAAA` and optionally `SSHFP` records) and is configured from that service's **Checks** tab.

## What it checks

| Rule | Verifies | Severity |
|------|----------|----------|
| `ssh.tcp_reachable` | Every probed (address, port) pair accepts a TCP connection. | Critical |
| `ssh.handshake` | The SSH banner exchange and KEXINIT parse succeed on every reachable endpoint. | Critical |
| `ssh.protocol_version` | Every endpoint advertises SSH-2 and rejects legacy SSH-1. | Critical |
| `ssh.banner_software` | Flags servers whose banner is not a recognised OpenSSH build. | Info |
| `ssh.known_vulnerabilities` | Matches the advertised OpenSSH version against a curated CVE catalog (regreSSHion, Terrapin, etc.). | Critical |
| `ssh.host_key_strength` | Flags host keys below the accepted minimum size (e.g. RSA < 2048 bits). | Critical |
| `ssh.kex_algorithms` | Flags weak or broken key-exchange algorithms. | Critical |
| `ssh.host_key_algorithms` | Flags weak or deprecated host-key algorithms (ssh-rsa/SHA-1, ssh-dss…). | Critical |
| `ssh.cipher_algorithms` | Flags weak or broken symmetric ciphers (CBC, 3DES, RC4…). | Critical |
| `ssh.mac_algorithms` | Flags weak MAC algorithms (SHA-1, non-ETM…). | Critical |
| `ssh.strict_kex` | The server advertises the strict-KEX marker (CVE-2023-48795 Terrapin mitigation). | Warning |
| `ssh.preauth_compression` | Flags servers offering pre-authentication zlib compression. | Info |
| `ssh.auth_methods` | Reviews advertised authentication methods (password exposure, public-key availability). | Warning |
| `ssh.sshfp_alignment` | Compares published SSHFP records against observed host keys (match, missing, mismatch). | Critical |
| `ssh.sshfp_hash` | Flags SSHFP record sets that only publish SHA-1 (type 1) fingerprints. | Warning |

CVE matching covers, among others, regreSSHion (CVE-2024-6387), the ssh-agent PKCS#11 RCE (CVE-2023-38408), Terrapin (CVE-2023-48795), and several older username-enumeration and command-injection issues.

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Ports (`ports`) | Comma-separated extra TCP ports to probe. Port 22 is always probed. | (empty) |
| Per-endpoint probe timeout (ms) (`probeTimeoutMs`) | Maximum time for dial + banner + KEXINIT + handshake on a single endpoint. | 10000 |
| Enumerate authentication methods (`includeAuthProbe`) | Open a second connection with a dummy user to discover advertised auth methods. | true |

## In happyDomain

This is a service-level checker: configure it from the **Checks** tab of the *Server* service on the relevant subdomain. Its SSHFP rules cross-reference the `SSHFP` records published in your zone, so keeping those records in sync with the server's host keys improves the result. For the general workflow of configuring and reading checks, see {{< relref "/pages/checks" >}}.
