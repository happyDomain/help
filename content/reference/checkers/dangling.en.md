---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Dangling records
description: "Scans a zone for CNAME/MX/SRV/NS records whose targets resolve to NXDOMAIN or whose external domain has expired and could be re-registered."
weight: 140
---

The **Dangling subdomains** checker scans a zone for pointer records (`CNAME`, `MX`, `SRV`, `NS`) whose targets have gone stale: they resolve to NXDOMAIN, or their external registrable domain has expired, is in `pendingDelete`, or was recently re-registered. This is the subdomain-takeover attack class popularised in 2017, where institutions ended up serving hostile content from CNAMEs pointing at decommissioned third-party services after attackers re-registered the lapsed targets.

This is a **zone-level** checker: it needs the full zone content and runs a single pass over it, consolidating findings by owner rather than producing one result per record.

## What it checks

The checker walks every service in the working zone and extracts pointer records from `CNAME`, special CNAME, `MX`, unknown `SRV` and orphan (bare `NS`/`CNAME`/`MX`) bodies. For each `(owner, type, target)` triple it classifies the target as in-zone or external (relative to the zone's registrable domain), performs a single time-bounded DNS resolution to detect immediate breakage, and publishes a discovery entry so a companion `domain_expiry` checker can run RDAP/WHOIS on external targets.

It emits one finding per impacted owner, ranked by descending severity:

| Signal | Severity | Source |
|--------|----------|--------|
| Target NXDOMAIN | Critical | Local DNS resolution |
| Target SERVFAIL | Warning | Local DNS resolution |
| Target NOERROR with empty answer | Info | Local DNS resolution |
| Registrable domain expired | Critical | `whois` related observation |
| Registrable status `pendingDelete` / `redemptionPeriod` | Critical | `whois` related observation |
| Registrable domain registered within the last 90 days | Warning | `whois` related observation |

{{% notice style="info" title="WHOIS signals need a companion checker" %}}
The DNS-resolution signals (NXDOMAIN, SERVFAIL, empty answer) work on their own. The WHOIS-driven signals (expired, `pendingDelete`, recently registered) only fire when the host's `domain_expiry` checker subscribes to this checker's external-target discovery entries and publishes a per-target `whois` observation. Without that wiring, the checker still works as a DNS-only dangling detector.
{{% /notice %}}

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Skip live DNS resolution | When set, the checker only reports the static structure of pointer records (offline analysis), without resolving targets. | `false` |

## In happyDomain

Enable this checker on the domain from the {{< relref "/pages/checks" >}} view; the domain name and zone content are filled in automatically. Because it is zone-scoped, it runs over the whole zone in a single pass.

Related checkers: {{< relref "/reference/checkers/alias" >}} validates the structure of individual alias chains, and {{< relref "/reference/checkers/domain-expiry" >}} watches your own domains' expiry — the same WHOIS machinery that powers this checker's external-target signals.
