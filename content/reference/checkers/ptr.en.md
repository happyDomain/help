---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: PTR records
description: "Validates reverse DNS for an IP: PTR presence, target syntax, forward resolution and Forward-Confirmed Reverse DNS (FCrDNS)."
weight: 120
---

The **PTR / Reverse DNS** checker verifies that an IP address has a correct, usable reverse-DNS record. A PTR maps an IP back to a hostname; mail servers, SSH daemons and many logging tools rely on it, and a missing or inconsistent PTR is one of the most common reasons outgoing mail is rejected.

This is a **service-level** checker: it runs on a `PTR` service and is configured from that service's own **Checks** tab. It locates the reverse zone (under `in-addr.arpa` or `ip6.arpa`), queries the authoritative servers, and inspects what they actually serve — both for IPv4 and IPv6 addresses.

## What it checks

The checker runs a chain of rules, from structural sanity through query success to target hygiene and Forward-Confirmed Reverse DNS (FCrDNS).

| Rule | What it verifies | Severity |
|------|------------------|----------|
| `ptr.in_reverse_arpa` | The PTR owner lies under `in-addr.arpa` or `ip6.arpa`. | Critical |
| `ptr.owner_decodable` | The reverse-arpa owner name decodes back to an IP address. | Critical |
| `ptr.reverse_zone_located` | The reverse zone serving the owner can be located (SOA found). | Critical |
| `ptr.query_succeeded` | The PTR query returns NOERROR from the authoritative servers. | Critical |
| `ptr.record_present` | At least one PTR record is served at the owner name. | Critical |
| `ptr.single_record` | Flags more than one PTR on the same IP (RFC 1912 §2.1 recommends exactly one). | Warning |
| `ptr.declared_match` | The PTR target served authoritatively matches the target declared in happyDomain. | Critical |
| `ptr.target_syntax_valid` | The PTR target is a syntactically valid hostname (RFC 952/1123). | Critical |
| `ptr.generic_hostname` | Flags PTR targets that embed the IP or match common ISP auto-generated patterns. | Warning |
| `ptr.target_resolves` | The PTR target resolves to at least one A or AAAA record. | Critical / Warning |
| `ptr.fcrdns_match` | The PTR target's A/AAAA resolves back to the original IP (FCrDNS). | Critical / Warning |
| `ptr.ipv6` | Reports whether the PTR concerns an IPv6 (`ip6.arpa`) address, and that a PTR is present for it. | Critical |
| `ptr.ttl_hygiene` | The PTR TTL is at or above the configured minimum. | Warning |

The `ptr.target_resolves` and `ptr.fcrdns_match` rules are critical by default but drop to a warning when **Require forward-confirmed reverse DNS** is turned off.

{{% notice style="info" title="FCrDNS, the round-trip rule" %}}
Forward-Confirmed Reverse DNS means the chain rounds back to itself: the IP's PTR points to a hostname, and that hostname's A/AAAA includes the original IP. Mail servers reject connections from IPs that fail this round-trip, so leave **Require forward-confirmed reverse DNS** enabled for any host that sends mail.
{{% /notice %}}

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Require forward-confirmed reverse DNS (FCrDNS) | When enabled, a PTR whose target does not resolve back to the original IP is critical; otherwise a warning. | `true` |
| Allow multiple PTR records on the same IP | When disabled, more than one PTR at the same owner is flagged as a warning (RFC 1912 §2.1). | `false` |
| Minimum PTR TTL (seconds) | PTR records with a TTL below this threshold are flagged as a warning. | `300` |
| Flag generic-looking PTR hostnames | When enabled, PTR targets embedding the dotted IP or matching common ISP auto-generated patterns warn. | `true` |

## In happyDomain

Add the PTR service to the subdomain holding the reverse record, then enable this checker from that service's **Checks** tab. See {{< relref "/pages/checks" >}} for configuring and scheduling checks. The reverse zone, the PTR record and the declared target are filled in automatically from the service.

For the forward side of alias and hostname resolution, see the related {{< relref "/reference/checkers/alias" >}} checker.
