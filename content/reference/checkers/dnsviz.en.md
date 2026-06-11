---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: DNSViz
description: "Run DNSViz against a domain and walk the whole DNSSEC chain from the root down to the leaf, surfacing every error and warning at the exact zone where it occurs."
weight: 340
---

The **DNSViz** checker (named *DNSSEC (DNSViz)* in happyDomain) assesses a domain's DNSSEC validation chain by driving [DNSViz](https://github.com/dnsviz/dnsviz). It analyses the queried name **and every ancestor up to the root**, so a recursive DNSSEC failure can be located at the exact level, and the exact record, where it broke.

This checker is **domain-level**: it concerns the domain and its delegation chain rather than a single service.

## How it works

DNSViz is run externally (a Python tool packaged alongside the checker), and happyDomain delegates collection to that endpoint. For each run it effectively performs:

```
dnsviz probe -A . <ancestors…> <domain> | dnsviz grok -t <root-trust-anchor>
```

Every link of the chain (root → TLD → intermediates → leaf) is listed explicitly so the full analysis appears in the report. `dnsviz grok` is given a BIND-format DNSKEY trust anchor for the root zone; without it, the root has no parent to chain against and stays classified at the DNS level (`NOERROR`) instead of DNSSEC `SECURE`.

The output is parsed: per-zone errors and warnings are walked out of the nested record tree and turned into individual states tagged with the JSON path where they were found. A curated catalog of common DNSSEC failures (broken chain, expired RRSIG, DS digest mismatch, deprecated algorithm…) is matched against the findings to drive a "Fix these first" section in the HTML report.

{{% notice style="info" title="Scope" %}}
This checker reports exactly what DNSViz reports. NSEC/NSEC3 zone-walk hardening and NSEC3PARAM iteration policy (RFC 9276) are out of scope here and handled by the dedicated DNSSEC checker — see {{< relref "/reference/checkers/dnssec" >}}.
{{% /notice %}}

## What it checks

| Rule | What it verifies |
|---|---|
| `dnsviz_overall_status` | DNSViz status of the queried domain (SECURE / INSECURE / BOGUS / INDETERMINATE). |
| `dnsviz_per_zone_status` | One state per zone in the chain (root, TLD, intermediates, leaf). |
| `dnsviz_zone_errors` | Every error reported by DNSViz, scoped to the zone where it was found. |
| `dnsviz_zone_warnings` | Every warning reported by DNSViz, scoped to the zone where it was found. |
| `dnsviz_common_failures` | Pattern-matches the findings against a catalog of common DNSSEC failures. |

Statuses map as follows: `SECURE` → OK; `BOGUS` → Critical; `INDETERMINATE` → Warning; `INSECURE`, `NON_EXISTENT` and a plain `NOERROR` (resolves but unsigned) → Info.

## Options

| Option | Meaning | Default |
|---|---|---|
| Probe timeout (`probeTimeoutSeconds`, admin) | Hard timeout for the `dnsviz probe` invocation; the recursive walk can be slow on some zones. | 120 |
| Domain name (`domain_name`) | Domain to analyse. | auto-filled |

The DNSViz runtime itself requires `dnsviz` on the host's `PATH` and a BIND-format root DNSKEY trust anchor file (auto-detected from the system's `dnssec-root` / `dns-root-data` package, or pointed at explicitly). The packaged container image bundles these so the trust anchor is in place out of the box.

## In happyDomain

Enable this checker for the domain from the **Checks** view. See {{< relref "/pages/checks" >}} for scheduling and reading checks.

DNSViz and {{< relref "/reference/checkers/dnssec" >}} are complementary: DNSViz validates the end-to-end chain, while the DNSSEC checker covers NSEC/NSEC3 hardening. A second opinion on the same chain is available from {{< relref "/reference/checkers/zonemaster" >}}.
