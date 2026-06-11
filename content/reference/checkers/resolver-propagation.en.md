---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Resolver propagation
description: "Probes a worldwide catalog of public recursive resolvers across several transports and regions, then compares their answers to the zone's authoritative servers."
weight: 100
---

The **Resolver propagation** checker measures how a zone is seen from the *outside*, across the public internet. It probes a curated catalog of public recursive resolvers (Cloudflare, Google, Quad9, OpenDNS, Yandex, regional ISPs and more) over several transports (UDP, TCP, DoT, DoH) and from multiple regions, then compares their answers both to each other and to the zone's authoritative name servers. This surfaces propagation gaps, regional splits, `SOA` serial drift, stale caches, DNSSEC validation failures, `SERVFAIL`/`NXDOMAIN` inconsistencies and resolver filtering.

This checker is **service-level**: it targets an *Origin* or *NS-only Origin* service (`abstract.Origin`, `abstract.NSOnlyOrigin`) and is configured from that service's **Checks** tab.

## What it checks

| Finding code | What it checks | Severity |
|---|---|---|
| `resolver_propagation.selection` | The current option set selects at least one public resolver. | Critical |
| `resolver_propagation.reachable` | At least one selected resolver answered a query. | Critical |
| `resolver_propagation.latency` | Resolvers that are unreachable or whose average response time exceeds the threshold. | Warning |
| `resolver_propagation.filtered_hit` | Filtered resolvers returning a different answer than the consensus (typical blocklist behaviour). | Info |
| `resolver_propagation.consensus` | Public resolvers agree on a single answer for each probed RRset. | Warning |
| `resolver_propagation.matches_authoritative` | The public consensus matches the answer served by the zone's authoritative servers. | Critical |
| `resolver_propagation.nxdomain` | RRsets for which some resolvers return `NXDOMAIN` while others return `NOERROR`. | Critical |
| `resolver_propagation.servfail` | RRsets for which any resolver returns `SERVFAIL` (usually DNSSEC or reachability failure). | Critical |
| `resolver_propagation.regional_split` | Regions where every resolver agrees on an answer that differs from the global consensus. | Warning |
| `resolver_propagation.serial_drift` | Disagreement on the `SOA` serial across unfiltered resolvers. | Warning |
| `resolver_propagation.stale_cache` | Resolvers still serving an `SOA` serial below the one saved by happyDomain. | Info |
| `resolver_propagation.dnssec` | Validating resolvers successfully validate the zone's DNSSEC chain. | Critical |

## Options

| Option | Meaning | Default |
|---|---|---|
| `recordTypes` | Comma-separated RR types to probe at every owner. Empty = derive from the working zone (SOA/NS at the apex plus the RR types actually defined on each owner). | _derived from zone_ |
| `subdomains` | Comma-separated owner names to probe in addition to the apex (e.g. `www,mail,@`). Empty = apex only. | `www` |
| `includeFiltered` | Probe filtering resolvers (malware/family/adblock). Their answers diverge by design; enable only when diagnosing a blocklist hit. | `false` |
| `region` | Restrict to a region: `all`, `global`, `na`, `eu`, `asia`, `ru`, `me`. | `all` |
| `transports` | Comma-separated transports to probe: `udp`, `tcp`, `dot`, `doh`. Encrypted transports are only used where published. | `udp` |
| `resolverAllowlist` | Comma-separated resolver IDs or IPs to probe exclusively (e.g. `cloudflare,google,9.9.9.9`). Empty = catalog selection. | _(empty)_ |
| `latencyThresholdMs` | Resolvers averaging above this value emit an info finding. | `500` |
| `runTimeoutSeconds` | Hard wall-clock budget for one propagation run. Slower resolvers report as unreachable. | `30` |

## In happyDomain

Enable the Resolver propagation checker from the **Checks** tab of an Origin service. See {{< relref "/pages/checks" >}} for the full workflow. This checker is the outward-facing counterpart to {{< relref "/reference/checkers/authoritative-consistency" >}}, which examines the authoritative servers directly; running both gives you the picture from the origin and from the resolvers that query it.
