---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Name-server restrictions
description: "Probes every authoritative name server of a zone to confirm it is properly locked down: zone transfers refused, no open recursion, and RFC 8482 handling of ANY."
weight: 90
---

The **Name-server restrictions** checker verifies that the authoritative name servers of a zone are properly locked down. For each declared name server it resolves the host name, then runs a set of DNS probes against every returned IPv4 and IPv6 address (IPv6 targets are skipped gracefully when the host has no IPv6 connectivity). The goal is to catch common misconfigurations that leak data or turn a name server into an abuse vector: open zone transfers, open recursion, and unbounded `ANY` responses.

This checker is **service-level**: it targets an *Origin* or *NS-only Origin* service (`abstract.Origin`, `abstract.NSOnlyOrigin`) and is configured from that service's **Checks** tab.

## What it checks

Each rule emits one finding per probed name-server address, with a stable `code`.

| Rule | Verifies | Severity on failure |
|---|---|---|
| `ns_resolution` | Every NS host name declared in the delegation resolves to at least one IP address. | Critical |
| `ns_axfr_refused` | `AXFR` zone transfers are refused by every authoritative name server. | Critical |
| `ns_ixfr_refused` | `IXFR` zone transfers are refused by every authoritative name server. | Warning |
| `ns_no_recursion` | Authoritative name servers do not advertise recursion (RA bit unset). | Warning |
| `ns_any_handled` | `ANY` queries are handled per RFC 8482 (HINFO or a minimal answer rather than the full zone contents). | Warning |
| `ns_is_authoritative` | Name servers answer authoritatively (AA bit set) for the zone. | Info |

{{% notice style="info" title="Why these matter" %}}
An open `AXFR` lets anyone download the entire zone, exposing your internal naming. Open recursion turns your authoritative server into an amplification relay and cache-poisoning target. Unbounded `ANY` responses are a classic amplification vector that RFC 8482 was written to neutralise.
{{% /notice %}}

## Options

This checker has no user-tunable options: it runs a fixed set of probes against each resolved name-server address.

## In happyDomain

Enable the Name-server restrictions checker from the **Checks** tab of an Origin service. See {{< relref "/pages/checks" >}} for the full workflow. For the broader health and agreement of those same authoritative servers, see {{< relref "/reference/checkers/authoritative-consistency" >}}.
