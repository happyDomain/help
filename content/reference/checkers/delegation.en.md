---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Delegation
description: "Audits a zone's delegation: NS consistency between parent and child, glue correctness, DS/DNSKEY hand-off, reachability and authoritativeness of each delegated server."
weight: 70
---

The **Delegation** checker audits how a zone is delegated from its parent. It cross-examines the parent zone and the child name servers to confirm that the hand-off is coherent: that the parent and child agree on the `NS` set, that glue records are correct, that the DNSSEC `DS`/`DNSKEY` chain lines up, and that every delegated server is reachable and actually authoritative for the zone.

This checker is **service-level**: it targets a *Delegation* service (`abstract.Delegation`) published on a subdomain, and is configured from that service's own **Checks** tab.

## What it checks

Each rule emits a stable finding `code` so results can be matched deterministically.

### Name-server count and parent discovery

| Finding code | What it verifies |
|---|---|
| `delegation_too_few_ns` | The zone declares at least `minNameServers` `NS` records (RFC 1034 recommends ≥ 2). |
| `delegation_no_parent_ns` | The parent zone and its authoritative name servers can be discovered. |
| `delegation_parent_query_failed` | Each parent name server answers the `NS` query for the delegated zone. |
| `delegation_parent_tcp_failed` | Each parent name server is reachable over TCP (RFC 7766). |

### NS and glue at the parent

| Finding code | What it verifies |
|---|---|
| `delegation_ns_mismatch` | The `NS` RRset at the parent matches the `NS` set declared by the service. |
| `delegation_missing_glue` | In-bailiwick name servers have glue (`A`/`AAAA`) at the parent. |
| `delegation_unnecessary_glue` | Out-of-bailiwick name servers do not carry unnecessary glue. |

### DNSSEC hand-off

| Finding code | What it verifies |
|---|---|
| `delegation_ds_query_failed` | The `DS` RRset can be queried from the parent name servers. |
| `delegation_ds_mismatch` | The `DS` RRset at the parent matches the `DS` set declared by the service. |
| `delegation_ds_missing` | `DS` records are present at the parent when DNSSEC is expected (gated by `requireDS`). |
| `delegation_ds_rrsig_invalid` | The `DS` RRset is covered by a valid `RRSIG` at the parent. |
| `delegation_dnskey_query_failed` | The `DNSKEY` RRset can be queried from each child name server. |
| `delegation_dnskey_no_match` | At least one child `DNSKEY` matches a `DS` digest published at the parent. |

### Child reachability and authoritativeness

| Finding code | What it verifies |
|---|---|
| `delegation_ns_unresolvable` | Each declared name server name resolves to at least one address. |
| `delegation_unreachable` | Each child name server answers DNS queries on its advertised addresses. |
| `delegation_lame` | Each child name server is authoritative for the zone (no lame delegation). |
| `delegation_no_authoritative_answer` | Each child name server sets the AA flag in its answers for the zone. |
| `delegation_tcp_failed` | Each child name server answers over TCP (gated by `requireTCP`). |
| `delegation_soa_serial_drift` | The `SOA` serial is consistent across all child name servers. |
| `delegation_ns_drift` | The `NS` RRset returned by each child matches the `NS` RRset at the parent. |
| `delegation_glue_mismatch` | Glue addresses at the child match those at the parent (gated by `allowGlueMismatch`). |

## Options

| Option | Meaning | Default |
|---|---|---|
| `requireDS` | When enabled, missing `DS` records at the parent are treated as critical (otherwise informational). | `false` |
| `requireTCP` | When enabled, name servers that fail to answer over TCP are reported as critical (otherwise warning). | `true` |
| `minNameServers` | Below this count, the delegation is reported as a warning (RFC 1034 recommends at least 2). | `2` |
| `allowGlueMismatch` | When disabled, glue/address mismatches between parent and child are reported as critical. | `false` |

## In happyDomain

Enable the Delegation checker from the **Checks** tab of a Delegation service. See {{< relref "/pages/checks" >}} for the full workflow. For consistency *between* the authoritative servers of the apex itself (rather than the parent/child hand-off), see {{< relref "/reference/checkers/authoritative-consistency" >}}; for the DNSSEC hygiene of the signed zone, see {{< relref "/reference/checkers/dnssec" >}}.
