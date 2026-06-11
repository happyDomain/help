---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: XMPP
description: "Probes a domain's XMPP deployment: SRV discovery, reachability, STARTTLS, SASL mechanisms and federation authentication."
weight: 290
---

The **XMPP** checker probes a domain's XMPP (Jabber) deployment end-to-end, much like [xmpp.net](https://xmpp.net/) does: it discovers the relevant SRV records, opens a stream to each endpoint, negotiates STARTTLS, inspects the offered SASL mechanisms and confirms that server-to-server federation can authenticate.

This is a **service-level** checker. It applies to services of type **XMPP** and is configured from that service's own **Checks** tab. It probes the four standard service names (`_xmpp-client._tcp`, `_xmpp-server._tcp`, `_xmpps-client._tcp`, `_xmpps-server._tcp`), the legacy `_jabber._tcp`, and falls back to `<domain>:5222` / `:5269` when no SRV record is published.

{{% notice style="info" title="TLS posture is checked separately" %}}
Certificate chain, hostname (SAN) match, expiry and cipher posture are **out of scope** here: a dedicated {{< relref "/reference/checkers/tls" >}} checker handles them. The XMPP checker only confirms that STARTTLS completes, records the negotiated TLS version and cipher for context, and folds the downstream TLS findings back onto the XMPP service report through the `xmpp.tls_quality` rule.
{{% /notice %}}

## What it checks

| Rule | What it verifies | Severity |
|------|------------------|----------|
| `xmpp.srv_c2s` | Client-to-server SRV records (`_xmpp-client` / `_xmpps-client` / `_jabber`) are published and resolvable. | Critical |
| `xmpp.srv_s2s` | Server-to-server SRV records (`_xmpp-server` / `_xmpps-server`) are published and resolvable. | Critical |
| `xmpp.c2s_reachable` | At least one client-to-server endpoint accepts TCP and completes TLS. | Critical |
| `xmpp.s2s_reachable` | At least one server-to-server endpoint accepts TCP and completes TLS. | Critical |
| `xmpp.starttls_required` | STARTTLS is advertised and required on every reachable c2s/s2s endpoint. | Critical |
| `xmpp.sasl_mechanisms` | The c2s SASL offer is sound (SCRAM present, no password-equivalent PLAIN-only). | Critical |
| `xmpp.s2s_dialback` | Server-to-server endpoints advertise dialback or SASL EXTERNAL after TLS (federation auth). | Critical |
| `xmpp.ipv6_reachable` | Flags deployments reachable only over IPv4. | Info |
| `xmpp.direct_tls` | Flags c2s deployments that do not publish XEP-0368 direct-TLS (`_xmpps-*`) SRV records. | Info |
| `xmpp.tls_quality` | Folds the downstream TLS checker findings (certificate chain, hostname match, expiry) onto the XMPP service. | Critical |

The probe also covers TCP reachability of A/AAAA targets, stream feature parsing and IPv4/IPv6 coverage, surfaced through the rules above and the HTML report.

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Domain | XMPP domain (JID domain) to test. Filled in automatically from the service. | (auto-filled) |
| Mode | Which side to probe: `c2s` (client-to-server), `s2s` (server-to-server), or `both`. | `both` |
| Per-endpoint timeout (seconds) | Time budget for each probed endpoint. | 10 |

## In happyDomain

Enable this checker from the **Checks** tab of an XMPP service; see {{< relref "/pages/checks" >}} for how to configure and schedule checks. The domain is filled in automatically from the service. For the certificate side of the same endpoints, pair it with the {{< relref "/reference/checkers/tls" >}} checker.
