---
date: 2026-06-11T09:00:00+02:00
title: Checkers
author: nemunaire
archetype: chapter
weight: 35
aliases:
    checkers
---

Checkers are the building blocks of happyDomain's monitoring system. Each one collects data about a domain, a zone or a service, evaluates it against a set of rules, and reports a clear status (OK, Warning, Critical or Error).

This chapter documents every checker shipped with happyDomain: what it verifies, the scope it applies to, the options you can tune, and the rules it evaluates. For the day-to-day workflow of configuring, scheduling and reading checks in the interface, see {{< relref "/pages/checks" >}}.

## Scopes

Every checker declares the scope it operates on:

- **Domain-level** — concerns the domain itself, independent of its DNS records (registration status, expiry, transfer lock…).
- **Zone-level** — needs the full zone content (DNSSEC validation, delegation consistency…).
- **Service-level** — targets a specific service published on a subdomain (HTTP, TLS, ping…), and is configured from that service's own **Checks** tab.

## Statuses

Checkers report one of the following statuses, in order of severity:

| Status | Meaning |
|--------|---------|
| **OK** | Everything is within acceptable parameters |
| **Info** | Informational finding, no action needed |
| **Warning** | A threshold is approaching; attention recommended |
| **Critical** | A threshold has been exceeded; action required |
| **Error** | The check itself failed (collection error, bad configuration) |
| **Unknown** | The check could not determine a result |

{{% children %}}
