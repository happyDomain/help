---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Reverse zone
description: "Inspects the PTR records of an in-addr.arpa or ip6.arpa reverse zone for FCrDNS, target resolvability, hostname syntax, generic names and TTL hygiene."
weight: 110
---

The **Reverse zone** checker inspects the `PTR` records of a reverse DNS zone (`in-addr.arpa` or `ip6.arpa`) and validates that they are well formed and consistent with forward DNS. It verifies Forward-Confirmed Reverse DNS (FCrDNS), that targets resolve and are syntactically valid host names, flags generic or auto-generated names and short TTLs, and catches multiple-`PTR`-per-IP violations (RFC 1912 §2.1). Correct reverse DNS matters in practice: mail servers and SSH endpoints routinely reject or downgrade connections from IPs without proper FCrDNS.

This checker is **zone-level**: it operates on the full content of a reverse zone (it applies to the domain and reads the whole zone).

## What it checks

| Finding code | What it verifies | Severity |
|---|---|---|
| `reverse_zone.is_reverse_arpa` | The zone is under `in-addr.arpa` or `ip6.arpa`. | Critical |
| `reverse_zone.has_ptrs` | The reverse zone declares at least one `PTR` record. | Warning |
| `reverse_zone.fcrdns` | Every `PTR` target's `A`/`AAAA` round-trips back to the original IP (Forward-Confirmed Reverse DNS). | Critical |
| `reverse_zone.target_resolves` | Every `PTR` target resolves to at least one `A` or `AAAA` record. | Critical |
| `reverse_zone.single_ptr_per_ip` | Flags IPs with multiple `PTR` records (RFC 1912 §2.1 recommends exactly one). | Warning |
| `reverse_zone.target_syntax` | Every `PTR` target is a syntactically valid host name. | Critical |
| `reverse_zone.generic_hostname` | Flags `PTR` targets that embed the IP or match common ISP auto-generated patterns. | Warning |
| `reverse_zone.ttl_hygiene` | Flags `PTR` records whose TTL is below the configured minimum. | Warning |
| `reverse_zone.truncated` | Reports when the zone has more `PTR`s than the configured cap allows to inspect. | Info |

## Options

| Option | Meaning | Default |
|---|---|---|
| `requireForwardMatch` | When enabled, a `PTR` whose target does not resolve back to the original IP is critical (otherwise warning). Mail and SSH servers require FCrDNS. | `true` |
| `allowMultiplePTR` | When disabled, more than one `PTR` at the same owner is reported as warning (RFC 1912 §2.1 recommends a single `PTR` per IP). | `false` |
| `minTTL` | `PTR` records with a TTL below this threshold (in seconds) are flagged as warning. | `300` |
| `flagGenericPTR` | When enabled, `PTR` targets that embed the dotted IP or match common ISP auto-generated patterns are flagged as warning. | `true` |
| `maxPTRsToCheck` | Caps the number of `PTR` records inspected per run, protecting the checker against very large reverse zones. | `1024` |

{{% notice style="info" title="Forward-Confirmed Reverse DNS" %}}
FCrDNS means the `PTR` target, looked up forward, resolves back to the original IP address. A `PTR` that points to a host whose `A`/`AAAA` does not include that IP fails the round-trip and is treated as a misconfiguration by many mail and SSH servers.
{{% /notice %}}

## In happyDomain

Enable the Reverse zone checker on a reverse (`in-addr.arpa` / `ip6.arpa`) domain from its **Checks** view. See {{< relref "/pages/checks" >}} for the full workflow.
