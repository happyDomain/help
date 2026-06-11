---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: DANE / TLSA
description: "Matches a service's TLSA records against the certificate chain actually presented by each endpoint, per RFC 6698."
weight: 160
---

The **DANE / TLSA** checker verifies that the `TLSA` records published for a service correctly pin the TLS certificate that the matching endpoint actually presents. DANE (DNS-based Authentication of Named Entities) lets a domain bind a certificate or public key to a name through DNS, secured by DNSSEC; this checker confirms the binding holds.

This is a **service-level** checker: it runs on a `TLSAs` service. It groups the declared TLSA records by `(port, proto, base)`, publishes one TLS endpoint discovery entry per endpoint so `checker-tls` probes it, then matches each TLSA against the observed certificate chain following RFC 6698.

## What it checks

| Rule | What it verifies / flags | Severity |
|------|--------------------------|----------|
| `dane.has_records` | At least one TLSA record is declared on the service. | — |
| `dane.dnssec_validated` | The TLSA records were fetched via a DNSSEC-validating resolver (AD bit set). | — |
| `dane.probe_available` | A TLS probe is available for every DANE endpoint so the chain can be compared. | — |
| `dane.handshake_ok` | The TLS handshake succeeds on every DANE endpoint. | — |
| `dane.records_match_chain` | At least one TLSA record matches the certificate chain presented by each endpoint. | — |
| `dane.pkix_chain_valid` | When usages 0 or 1 are published, the chain also validates against system trust roots. | — |
| `dane.usage_coherent` | Flags TLSA records whose declared usage does not match the chain slot they actually hash (e.g. usage 3 matching an intermediate). | — |

### How the TLSA fields are interpreted

- **Usage 0 (PKIX-TA) / 1 (PKIX-EE)** — the TLSA must match *and* the chain must validate against the public PKIX trust roots.
- **Usage 2 (DANE-TA) / 3 (DANE-EE)** — the TLSA itself acts as the trust anchor; PKIX validity is informational.
- **Selector** 0 (full certificate) or 1 (Subject Public Key Info), and **matching type** 0 (full) / 1 (SHA-256) / 2 (SHA-512), are matched against the chain slot implied by the usage.

{{% notice style="info" title="DANE relies on DNSSEC" %}}
DANE is only trustworthy when the TLSA records are served from a DNSSEC-signed zone and validated by the resolver. The `dane.dnssec_validated` rule checks that the records arrived with the AD (Authenticated Data) bit set; without DNSSEC, a TLSA record could be forged and the whole binding is meaningless.
{{% /notice %}}

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Probe timeout (ms) | Forwarded to `checker-tls` as the per-endpoint TLS probe timeout. | `checker-tls` default |
| STARTTLS override | A map keyed by `"<port>/<proto>"` overriding the STARTTLS application used when probing. Common STARTTLS ports (25, 110, 143, 389, 587, 5222, 5269) are auto-mapped; set this only for non-standard ports. | (auto) |

The domain, subdomain and TLSAs service are filled in automatically.

## In happyDomain

Add the TLSA service to the subdomain, then enable this checker from that service's **Checks** tab. See {{< relref "/pages/checks" >}} for configuring and scheduling checks. The checker publishes its endpoints for `checker-tls` to probe, so allow a probe cycle before the first match result appears.

Related checker: {{< relref "/reference/checkers/caa" >}} verifies, on the same certificates, that the issuing CA was authorized by the domain's CAA policy.
