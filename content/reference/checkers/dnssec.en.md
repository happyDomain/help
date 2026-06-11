---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: DNSSEC validation
description: "Audits the operational hygiene of a signed zone: algorithms, key sizes, signature freshness, NSEC/NSEC3 parameters and per-name-server consistency."
weight: 60
---

The **DNSSEC** checker audits the *operational hygiene* of a signed zone. It does not re-validate the cryptographic chain of trust end to end (that work is delegated to a dedicated DNSViz-based checker); instead it focuses on the policy and day-to-day operational aspects of signing: which algorithms and key sizes are in use, whether signatures are fresh and valid, how negative answers are denied (NSEC vs NSEC3), and whether every authoritative server serves a consistent view.

This checker is **domain-level**: it operates on the zone apex. From a bootstrap recursive resolver it discovers the apex name servers and looks up the parent `DS`, then queries each authoritative server directly.

## What it checks

The checker evaluates the following rules. Several of them are governed by the options described further down.

### Chain and key material

| Rule | Verifies | Severity |
|---|---|---|
| `dnssec_zone_signed` | The parent advertises a `DS` but the apex serves no `DNSKEY` (a broken signed zone). | Critical |
| `dnssec_dnskey_consistent` | Every authoritative server returns the same `DNSKEY` RRset. | Critical |
| `dnssec_dnskey_query_ok` | Every authoritative server answered the `DNSKEY` query. | Critical |
| `dnssec_ksk_present` | At least one `DNSKEY` carries the SEP bit (a Key Signing Key). | Critical |
| `dnssec_dnskey_count` | Warns when too many `DNSKEY`s are published, inflating responses and amplification potential. | Warning |

### Algorithms and key strength

| Rule | Verifies | Severity |
|---|---|---|
| `dnssec_algorithm_allowed` | No `DNSKEY` uses a forbidden algorithm or one outside the allowed list. | Critical |
| `dnssec_algorithm_modern` | Recommends ECDSAP256SHA256 (13) or Ed25519 (15) over RSA. | Warning |
| `dnssec_rsa_keysize` | RSA `DNSKEY`s reach the minimum modulus size. | Critical |

### Signatures

| Rule | Verifies | Severity |
|---|---|---|
| `dnssec_rrsig_present_dnskey` | The `DNSKEY` RRset is signed. | Critical |
| `dnssec_rrsig_present_soa` | The `SOA` RRset is signed. | Critical |
| `dnssec_rrsig_validity_window` | Every observed `RRSIG` is currently within its inception/expiration window. | Critical |
| `dnssec_rrsig_freshness` | Pre-emptive alert when `RRSIG`s are close to expiring (catches stuck signers). | Critical |

### Denial of existence (NSEC / NSEC3)

| Rule | Verifies | Severity |
|---|---|---|
| `dnssec_denial_uses_nsec3` | Warns when the zone uses bare NSEC, which makes the zone walkable (RFC 5155 / RFC 7129). | Warning |
| `dnssec_nsec3_iterations` | `NSEC3PARAM.Iterations` is at most the configured ceiling (RFC 9276 §3.1 recommends 0). | Critical |
| `dnssec_nsec3_salt_empty` | `NSEC3PARAM.SaltLength` is 0 (RFC 9276 §3.1: a salt buys no measurable protection). | Warning |
| `dnssec_nsec3_optout_only_when_signed_delegations` | Informational note when the OPT-OUT flag is set in a leaf zone. | Info |
| `dnssec_denial_consistent` | Every authoritative server uses the same denial-of-existence scheme. | Warning |

### TTL hygiene

| Rule | Verifies | Severity |
|---|---|---|
| `dnssec_dnskey_ttl_min` | Warns when the `DNSKEY` TTL is too short to be useful for caching. | Warning |

## Options

| Option | Meaning | Default |
|---|---|---|
| `nsec3IterationsMax` | RFC 9276 §3.1 ceiling on `NSEC3PARAM.Iterations`. Raise only if your signer cannot publish 0 yet. | `0` |
| `nsec3IterationsSeverity` | Severity when iterations exceed the ceiling. Set `crit` to enforce RFC 9276 strictly. | `warn` |
| `signatureFreshness` | Warn when the closest `RRSIG` expires in fewer than this many days. | `7` |
| `signatureFreshnessCrit` | Critical when the closest `RRSIG` expires in fewer than this many days. | `1` |
| `minRSAKeySize` | Minimum acceptable RSA modulus size, in bits. | `2048` |
| `requireSEP` | Require at least one `DNSKEY` with the SEP bit (KSK). | `true` |
| `dnskeyTTLMin` | Minimum `DNSKEY` TTL, in seconds; shorter TTLs hurt cacheability. | `3600` |

The administrator can also set a bootstrap `resolver` (`host:port`) used to discover the apex name servers and look up the parent `DS`; it defaults to `/etc/resolv.conf`.

{{% notice style="info" title="What this checker does not do" %}}
The DNSSEC checker does not verify the full cryptographic chain of trust from the root. For that end-to-end validation, use the DNSViz-based checker. This checker complements it by catching policy and operational problems (weak algorithms, expiring signatures, NSEC walkability) that a chain validation alone would not surface.
{{% /notice %}}

## In happyDomain

Enable the DNSSEC checker on a domain from its **Checks** view. See {{< relref "/pages/checks" >}} for the full workflow of adding, scheduling and reading checks. For delegation-side `DS`/`DNSKEY` hand-off, see the {{< relref "/reference/checkers/delegation" >}} checker.
