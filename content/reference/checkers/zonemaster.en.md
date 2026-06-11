---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Zonemaster
description: "Run the Zonemaster test suite against a domain through its JSON-RPC API and group the results by test category (delegation, consistency, DNSSEC…)."
weight: 350
---

The **Zonemaster** checker runs the [Zonemaster](https://zonemaster.net/) test suite against a domain and reports its findings grouped by test category. Zonemaster is a well-established DNS health and delegation validator; this checker drives it through its JSON-RPC API, stores the full results as an observation, and renders an HTML report grouped by module and severity.

This checker is **domain-level**: it evaluates the domain's delegation and zone content rather than a single service.

## How it works

For each run the checker submits the domain to a Zonemaster JSON-RPC endpoint (the official public API by default), waits for the test batch to complete, and stores every message Zonemaster returns. Each message carries a module, a test case and a Zonemaster severity (INFO / NOTICE / WARNING / ERROR / CRITICAL).

{{% notice style="info" title="Point only at a trusted Zonemaster instance" %}}
The Zonemaster API URL is an administrator option. Because the checker issues requests to whatever URL is configured and surfaces the responses, point it only at a Zonemaster instance you trust.
{{% /notice %}}

## What it checks

The checker maps each Zonemaster test module onto a category rule. Every rule emits a `<rule>.summary` state, plus one state per WARNING-or-worse message (so downstream consumers can match on stable codes); INFO and NOTICE messages are folded into the summary counts.

| Rule | What it covers |
|---|---|
| `zonemaster.dnssec` | DNSSEC tests (signatures, NSEC/NSEC3, DS/DNSKEY coherence). |
| `zonemaster.delegation` | Delegation tests (parent/child NS agreement, glue, referrals). |
| `zonemaster.consistency` | Consistency tests (SOA serial, NS set, zone content across servers). |
| `zonemaster.connectivity` | Connectivity tests (UDP/TCP reachability of authoritative servers, AS diversity). |
| `zonemaster.nameserver` | Nameserver tests (server behaviour, EDNS, unknown RR handling). |
| `zonemaster.syntax` | Syntax tests (domain name syntax, hostname legality). |
| `zonemaster.zone` | Zone tests (SOA values, MX presence, mandatory records). |
| `zonemaster.address` | Address tests (nameserver IP addresses, private/reserved ranges). |
| `zonemaster.basic` | Basic/system tests (initial reachability and fundamental requirements). |

Zonemaster severities are mapped onto happyDomain statuses: CRITICAL and ERROR → Critical; WARNING → Warning; NOTICE, INFO and DEBUG → Info. Each summary takes the worst status among its category's messages. When Zonemaster returns no message for a category, that rule reports Unknown (not tested).

## Options

| Option | Meaning | Default |
|---|---|---|
| Domain name to check (`domainName`) | Domain submitted to Zonemaster. **Required.** | auto-filled |
| Profile (`profile`) | Zonemaster profile name to run the tests under. | `default` |
| Result language (`language`, per user) | Language of the returned messages (`en`, `fr`, `de`, `es`, `sv`, `da`, `fi`, `nb`, `nl`, `pt`). | `en` |
| Zonemaster API URL (`zonemasterAPIURL`, admin) | JSON-RPC endpoint to query. | `https://zonemaster.net/api` |

## In happyDomain

Enable this checker for the domain from the **Checks** view. See {{< relref "/pages/checks" >}} for scheduling and reading checks.

Zonemaster overlaps in part with {{< relref "/reference/checkers/dnsviz" >}} on DNSSEC, but covers a broader range of delegation, connectivity and zone-content tests. For NSEC/NSEC3 hardening specifically, see {{< relref "/reference/checkers/dnssec" >}}.
