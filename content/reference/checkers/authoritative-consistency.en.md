---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Authoritative consistency
description: "Probes every authoritative name server of a zone and verifies they agree with each other and with the parent on NS, SOA, reachability, EDNS0 and authoritativeness."
weight: 80
---

The **Authoritative consistency** checker probes every authoritative name server of a zone and verifies that they agree — with one another and with the parent delegation. Where the {{< relref "/reference/checkers/delegation" >}} checker focuses on the parent/child hand-off, this checker concentrates on the *apex itself*: do all the servers serve the same `NS` and `SOA`, are they all reachable over UDP and TCP, do they support EDNS0, do they answer authoritatively, and how fast do they respond?

This checker is **service-level**: it targets an *Origin* or *NS-only Origin* service (`abstract.Origin`, `abstract.NSOnlyOrigin`) and is configured from that service's **Checks** tab.

## What it checks

Each rule emits a finding code. Several severities depend on the options below.

| Finding code | Default severity | Condition |
|---|---|---|
| `authoritative_consistency_no_ns` | Critical | No name servers could be discovered (declared list empty and parent query returned nothing). |
| `authoritative_consistency_too_few_ns` | Warning | Fewer name servers declared than `minNameServers` (RFC 1034 recommends at least 2). |
| `authoritative_consistency_parent_query_failed` | Warning | The parent delegation query failed (network error, REFUSED…). |
| `authoritative_consistency_parent_drift` | Warning | The parent's `NS` RRset does not match the `NS` declared in the service. |
| `authoritative_consistency_ns_unresolvable` | Critical | A declared name server has no `A` or `AAAA` record. |
| `authoritative_consistency_ns_udp_failed` | Critical | A name server did not answer any SOA query over UDP/53. |
| `authoritative_consistency_ns_tcp_failed` | Critical with `requireTCP`, else Warning | A name server did not answer over TCP/53 (required by RFC 7766 and DNSSEC). |
| `authoritative_consistency_lame` | Critical | A name server answered without the AA bit for the zone (lame delegation). |
| `authoritative_consistency_no_soa` | Critical | A name server is authoritative but returned no `SOA`. |
| `authoritative_consistency_edns_unsupported` | Warning | A name server drops or mishandles EDNS0 queries (RFC 6891). |
| `authoritative_consistency_slow_ns` | Info | A name server's response time exceeded `latencyThresholdMs`. |
| `authoritative_consistency_serial_drift` | Warning | Authoritative servers disagree on the `SOA` serial (zone not fully propagated). |
| `authoritative_consistency_serial_stale_vs_saved` | Warning | The serial saved in happyDomain is newer than what the servers publish (likely un-pushed change). |
| `authoritative_consistency_serial_ahead_of_saved` | Info | The servers publish a serial newer than the saved one (out-of-band change). |
| `authoritative_consistency_soa_fields_drift` | Warning | Servers disagree on `SOA` fields (MNAME, RNAME, refresh, retry, expire, minimum). |
| `authoritative_consistency_ns_rrset_drift` | Warning | Servers disagree on the `NS` RRset they publish at the apex. |
| `authoritative_consistency_ns_rrset_mismatch_config` | Warning | The published `NS` RRset does not match the `NS` declared in the service. |

## Options

| Option | Meaning | Default |
|---|---|---|
| `requireTCP` | When enabled, a server that fails over TCP is critical (otherwise warning). TCP/53 is required by RFC 7766 and DNSSEC. | `true` |
| `checkEDNS` | Probe each name server for EDNS0 (RFC 6891). Servers that mishandle EDNS0 break DNSSEC and large answers. | `true` |
| `checkLatency` | Measure response time of every name server and warn on slow responders. | `true` |
| `latencyThresholdMs` | Response times above this value trigger a slow-server warning. | `500` |
| `useParentNS` | Query the parent for the delegation `NS` RRset and compare it to the service's declared name servers. | `true` |
| `warnOnStaleSaved` | Warn when the saved `SOA` serial is newer than what authoritative servers publish. | `true` |
| `minNameServers` | Below this count, a warning is emitted (RFC 1034 recommends at least 2). | `2` |

## In happyDomain

Enable the Authoritative consistency checker from the **Checks** tab of an Origin service. See {{< relref "/pages/checks" >}} for the full workflow. To compare what *recursive resolvers around the world* see against the authoritative answer, pair it with {{< relref "/reference/checkers/resolver-propagation" >}}.
