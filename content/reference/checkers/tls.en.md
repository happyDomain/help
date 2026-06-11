---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: TLS posture
description: "Dials every TLS endpoint discovered for a domain, completes the handshake (with STARTTLS upgrade where needed) and audits certificate chain, hostname coverage, expiry and protocol strength."
weight: 170
---

The **TLS** checker (internal name `TLS`) evaluates the transport-security posture of every TLS endpoint exposed by a domain. It does not scan ports on its own: it consumes **discovery entries** of type `tls.endpoint.v1` published by other service checkers (XMPP, SRV, CalDAV, CardDAV, SMTP…). For each discovered endpoint it performs a real TCP dial, an optional protocol-specific STARTTLS upgrade, and a full TLS handshake, then reports a per-endpoint posture.

**Scope:** domain-level. The checker runs against the whole domain and folds in the endpoints that every other service checker has advertised. A given endpoint is therefore only probed if some service checker (for example {{< relref "/reference/checkers/smtp" >}}) published it.

## What it checks

| Rule | Verifies | Severity |
|------|----------|----------|
| `tls.endpoints_discovered` | At least one TLS endpoint has been discovered for this target. | Info |
| `tls.reachability` | Every discovered endpoint accepts a TCP connection. | Critical |
| `tls.handshake` | The TLS handshake completes on every reachable endpoint. | Critical |
| `tls.starttls_advertised` | STARTTLS endpoints advertise the upgrade capability. | Critical |
| `tls.starttls_dialect_supported` | The discovered STARTTLS dialect is implemented by the checker. | Critical |
| `tls.peer_certificate_present` | The server presented a certificate during the handshake. | Critical |
| `tls.chain_validity` | The presented chain validates against the system trust store. | Critical |
| `tls.hostname_match` | The leaf certificate covers the probed hostname (SNI). | Critical |
| `tls.expiry` | Flags expired or soon-to-expire leaf certificates. | Critical |
| `tls.version` | Flags endpoints negotiating a TLS version below TLS 1.2. | Warning |
| `tls.cipher_suite` | Reports the cipher suite negotiated on each endpoint. | Info |
| `tls.enum.versions` | Flags endpoints that still accept TLS versions below TLS 1.2 (enumeration option). | Warning |
| `tls.enum.ciphers` | Flags endpoints accepting broken cipher suites (NULL, anonymous, EXPORT, RC4, 3DES) (enumeration option). | Warning |

STARTTLS upgrades are supported for SMTP/submission, IMAP, POP3, and XMPP (c2s and s2s). When a service checker marks an endpoint as requiring STARTTLS, the absence of the upgrade is reported as Critical; otherwise it is treated as opportunistic and reported as Warning.

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Per-endpoint probe timeout (ms) (`probeTimeoutMs`) | Maximum time allowed for dial + STARTTLS + TLS handshake on a single endpoint. | 10000 |
| Enumerate accepted TLS versions and cipher suites (`enumerateCiphers`) | When enabled, each direct-TLS endpoint is swept with one ClientHello per (version, cipher) pair to discover the exact set the server accepts. Adds roughly 50 handshakes per endpoint. | false |

The list of discovery entries is filled automatically from what other checkers publish and is not user-editable.

## In happyDomain

The TLS checker is a domain-level check: enable it from the domain's checks view. Because it works on endpoints discovered by other checkers, it pairs naturally with service-level checkers that publish TLS endpoints, such as {{< relref "/reference/checkers/smtp" >}}.

{{% notice style="info" title="TLS posture vs DANE" %}}
This checker validates the live certificate against the system trust store. Pinning the certificate in DNS through TLSA records is a separate concern, handled by the {{< relref "/reference/checkers/dane" >}} checker.
{{% /notice %}}

For the general workflow of configuring and reading checks, see {{< relref "/pages/checks" >}}.
