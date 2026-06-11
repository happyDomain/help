---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: SIP
description: "Probes a domain's SIP/VoIP deployment end-to-end: RFC 3263 NAPTR/SRV resolution, endpoint reachability over UDP/TCP/TLS, and a SIP OPTIONS ping with capability inspection."
weight: 270
---

The **SIP** checker probes a domain's SIP/VoIP deployment from its DNS records, following RFC 3263 resolution: NAPTR → SRV (`_sip._udp`, `_sip._tcp`, `_sips._tcp`) → A/AAAA. It tests reachability on every resolved `target:port` over UDP, TCP and TLS, then sends a raw SIP `OPTIONS` ping and inspects the reply (status line, `Server`/`User-Agent`, advertised `Allow` methods, round-trip time).

This checker is **service-level**: it targets a *SIP* service (`abstract.SIP`) published on a subdomain and is configured from that service's own **Checks** tab. When no SRV record is published, the checker falls back to `<domain>:5060` / `<domain>:5061`, with a visible info marker in the report.

{{% notice style="info" title="TLS posture is folded in, not duplicated" %}}
For the TLS handshake the checker uses `InsecureSkipVerify`: it confirms only that a TLS session can be established. Every `_sips._tcp` target is published as a `tls.endpoint.v1` discovery entry so the dedicated TLS checker can verify the certificate chain, hostname match, expiry and cipher posture. Those findings are folded back onto the SIP service page through the `sip.tls_quality` rule. See {{< relref "/reference/checkers/tls" >}}.
{{% /notice %}}

## What it checks

| Rule | What it verifies | Severity |
|---|---|---|
| `sip.srv_present` | `_sip._udp` / `_sip._tcp` / `_sips._tcp` SRV records are published and resolvable. | Critical |
| `sip.transport_diversity` | Modern SIP transports (TCP, and ideally TLS) are published alongside legacy UDP. | Warning |
| `sip.srv_targets_resolvable` | Every SRV target resolves to at least one A or AAAA address. | Critical |
| `sip.endpoint_reachable` | Every discovered SIP endpoint accepts a connection on its transport. | Critical |
| `sip.options_response` | Every reachable SIP endpoint answers `OPTIONS` with a 2xx response. | Critical |
| `sip.options_capabilities` | Reviews the `Allow` header advertised in `OPTIONS` replies (INVITE support, presence of `Allow`). | Warning |
| `sip.ipv6_coverage` | At least one SIP endpoint is reachable over IPv6. | Info |
| `sip.tls_quality` | Folds the downstream TLS checker findings (chain, hostname match, expiry) onto the SIP service. | Critical |

The checker performs, in order: the NAPTR lookup (`SIP+D2U`, `SIP+D2T`, `SIPS+D2T`), the SRV lookup for the three transports (with the `5060`/`5061` fallback), A/AAAA resolution of every SRV target, TCP connect / UDP send / TLS handshake, and the SIP `OPTIONS` request with its status, headers and `Allow` parsed.

## Options

| Option | Meaning | Default |
|---|---|---|
| SIP domain | The domain to test (auto-filled from the service scope). Required. | *(auto)* |
| Per-endpoint timeout (seconds) | Probe timeout for each endpoint. | `5` |
| Probe `_sip._udp` | Whether to probe the UDP transport. Disable if UDP is firewalled or the checker host cannot send UDP. | `true` |
| Probe `_sip._tcp` | Whether to probe the TCP transport. | `true` |
| Probe `_sips._tcp` (TLS) | Whether to probe the TLS transport. | `true` |

## In happyDomain

Enable the SIP checker from the **Checks** tab of a SIP service. The domain is filled in automatically. See {{< relref "/pages/checks" >}} for the full workflow, and {{< relref "/reference/checkers/tls" >}} for the certificate posture of the `_sips._tcp` endpoints.
