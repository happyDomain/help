---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Legacy records
description: "Scans a zone for DNS record types deprecated by the IETF and reports each one with its RFC reference and a migration suggestion."
weight: 330
---

The **Legacy DNS record types** checker scans a zone for record types the IETF has deprecated, and reports every occurrence with the relevant RFC reference and a concrete migration suggestion. Deprecated types clutter a zone and, in a few cases, are silently ignored by modern resolvers — so cleaning them up keeps the zone both tidy and unambiguous.

This is a **zone-level** checker: it walks every service in the working zone in a single pass and consolidates findings by record type, so the report shows one picture rather than one result per record.

## What it checks

A single rule, `legacy_records`, inspects each orphan record body for a deprecated type and groups the findings by severity. The default rule status is critical (the worst severity present bubbles to the top of the report).

| Severity | Record types | Why |
|----------|--------------|-----|
| Critical | `KEY`, `SIG`, `NXT` | RFC 3755: superseded by DNSKEY / RRSIG / NSEC; modern validators ignore them. |
| Warning | `SPF`, `A6`, `MD`, `MF` | RFC 7208 / RFC 6563 / RFC 973: replaced by TXT, AAAA, MX. |
| Info | `WKS`, `MB`, `MG`, `MR`, `MINFO`, `NULL`, `GPOS`, `NSAP`, `NSAP-PTR`, `X25`, `ISDN`, `RT`, `ATMA`, `EID`, `NIMLOC`, `SINK`, `NINFO`, `RKEY` | Experimental or historical (RFC 1035, 1183, 1706, 1712…); safe to delete. |

For each detected type the report names every owner where it appears, the RFC reason, the suggested replacement, and a concrete "how to fix" instruction. A clean zone produces a single OK state with the scan count; parse errors encountered during the scan are surfaced in a separate "skipped" section so a silent skip never masquerades as a clean pass.

{{% notice style="info" title="The SPF record type, not the SPF policy" %}}
The `SPF` *record type* (RFC 4408) was deprecated by RFC 7208 in favour of publishing the SPF policy in a `TXT` record. This checker flags the obsolete `SPF` record type, not your SPF policy itself — which remains valid and necessary when published as a `TXT` record.
{{% /notice %}}

## Options

This checker has no user-tunable options. The domain name and zone content are filled in automatically.

## In happyDomain

Enable this checker on the domain from the {{< relref "/pages/checks" >}} view; it runs over the whole zone in a single pass and needs no configuration. Use its findings as a clean-up checklist: each card tells you which record type to remove and what, if anything, to publish instead.
