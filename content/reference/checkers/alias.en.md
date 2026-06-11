---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Alias chain
description: "Walks a CNAME/DNAME/ALIAS chain and validates hop count, TTLs, target resolvability, apex coexistence and DNSSEC signing."
weight: 130
---

The **CNAME / DNAME / ALIAS chain** checker follows the alias chain of a name and verifies that it resolves cleanly: no loops, not too many hops, sane TTLs, a resolvable final target, no forbidden record coexistence, and a properly signed CNAME RRset when the zone uses DNSSEC.

This is a **service-level** checker: it runs on a `CNAME` (or special CNAME) service and is configured from that service's own **Checks** tab. Starting from the queried name it locates the zone apex, walks each CNAME/DNAME hop, and finally resolves the chain's target to A/AAAA records.

## What it checks

Each rule emits a finding code. Some severities are softened by the options below.

| Rule | What it verifies / flags | Severity |
|------|--------------------------|----------|
| `apex_lookup` | The zone apex (SOA) for the queried name can be located. | Critical |
| `chain_loop` | A CNAME/DNAME cycle in the resolution chain. | Critical |
| `chain_length` | The chain exceeds **Maximum chain length** hops. | Critical |
| `chain_query_error` | A DNS query fails while walking the chain (network error, timeout). | Warning |
| `chain_rcode` | A non-NOERROR response code mid-chain or on the final A/AAAA lookup. | Critical (mid-chain) / Warning (final) |
| `hop_ttl` | A CNAME/DNAME hop has a TTL below **Minimum target TTL**. | Warning |
| `cname_at_apex` | A CNAME exists at the zone apex, conflicting with SOA/NS (RFC 1912 §2.4). | Critical / Warning |
| `apex_flattening` | A/AAAA coexist with SOA/NS at the apex without a CNAME (provider-side ALIAS/ANAME flattening). | Info |
| `cname_coexistence` | Other RRsets (beyond A/AAAA) coexist at a CNAME owner, violating RFC 1034 §3.6.2 / RFC 2181 §10.1. | Critical / Warning at apex |
| `cname_dnssec` | The zone is DNSSEC-signed but the CNAME RRset lacks an RRSIG. | Critical |
| `target_resolvable` | The final target of the chain has no A or AAAA record. | Critical |
| `multiple_records` | An owner in the chain carries more than one CNAME/DNAME record (malformed). | Critical |

{{% notice style="info" title="Why a CNAME at the apex is a problem" %}}
A CNAME owner may carry no other record type, but the zone apex must always hold SOA and NS records. The two requirements are mutually exclusive, so a CNAME at the apex is invalid (RFC 1912 §2.4). Some providers work around this with server-side ALIAS/ANAME "flattening" that publishes plain A/AAAA at the apex; the `apex_flattening` rule recognises that pattern as intentional when **Recognize ALIAS/ANAME flattening** is enabled.
{{% /notice %}}

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Maximum chain length | Above this number of hops the chain is reported as critical. | `8` |
| Minimum target TTL | Hops with a TTL below this threshold (seconds) are flagged as a warning. | `60` |
| Allow CNAME at apex | When enabled, a CNAME at a zone apex and its coexistence violations are downgraded to warnings. RFC 1912 forbids this, so leaving it off is strongly recommended. | `false` |
| Recognize ALIAS/ANAME flattening | When enabled, providers serving A/AAAA at the apex (ALIAS/ANAME pseudo-records) are recognised as intentional and excused from coexistence violations. | `true` |

## In happyDomain

Add the CNAME service to the subdomain, then enable this checker from that service's **Checks** tab. See {{< relref "/pages/checks" >}} for configuring and scheduling checks. The parent domain and subdomain are filled in automatically.

Related checkers: {{< relref "/reference/checkers/dangling" >}} watches for alias targets that have become unregistered or NXDOMAIN, and {{< relref "/reference/checkers/ptr" >}} covers the reverse-DNS side.
