---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Kerberos
description: "Audits a Kerberos realm from its DNS records: SRV layout, KDC/kadmin/kpasswd reachability, an anonymous AS-REQ probe (realm, enctypes, pre-auth, clock skew), and an optional authenticated round-trip."
weight: 260
---

The **Kerberos** checker audits a Kerberos realm starting from its DNS records. From the realm name (derived in uppercase from the domain) and the SRV records grouped under the *Kerberos* service, it runs a series of **anonymous probes** and, when credentials are supplied, an optional **authenticated round-trip** — giving a combined picture of the realm's availability and security posture.

This checker is **service-level**: it targets a *Kerberos* service (`abstract.Kerberos`) published on a subdomain and is configured from that service's own **Checks** tab. It inspects the SRV layout (`_kerberos._tcp`, `_kerberos._udp`, `_kerberos-master._tcp`, `_kerberos-adm._tcp`, `_kpasswd._tcp`, `_kpasswd._udp`), forward-resolves every SRV target (A + AAAA), tests TCP reachability of each KDC/kadmin/kpasswd host and UDP reachability of the KDC via a real AS-REQ. The anonymous AS-REQ probe confirms the realm, reads the supported enctypes from `ETYPE-INFO2`, detects a PKINIT hint (`PA-PK-AS-REQ`) and measures clock skew.

{{% notice style="info" title="Credentials are forwarded to the KDC" %}}
When a principal and password are supplied, they are used once to obtain a TGT and then a TGS-REQ for the target service; they are forwarded to the KDC over the network and are never stored by the checker. Leave them blank to run anonymous probes only.
{{% /notice %}}

## What it checks

| Rule | What it verifies | Severity |
|---|---|---|
| `kerberos.srv_present` | At least one `_kerberos._tcp` / `_kerberos._udp` SRV record is published for the realm. | Critical |
| `kerberos.kdc_reachable` | At least one KDC endpoint (TCP/UDP 88) accepts a connection. | Critical |
| `kerberos.as_probe` | The anonymous AS-REQ probe received a sane reply (`KRB-ERROR` or `AS-REP`). | Critical |
| `kerberos.realm_match` | The KDC answers for the expected realm name. | Critical |
| `kerberos.preauth_required` | Flags KDCs that return an `AS-REP` without requiring pre-authentication (AS-REP roasting exposure). | Warning |
| `kerberos.clock_skew` | The KDC clock is within tolerance of the checker's clock. | Critical |
| `kerberos.enctypes` | Reviews the encryption types advertised by the KDC, flagging DES/RC4-only configurations. | Critical |
| `kerberos.kadmin_reachable` | Flags kadmin endpoints published via SRV but not reachable. | Warning |
| `kerberos.kpasswd_reachable` | Flags kpasswd endpoints published via SRV but not reachable. | Warning |
| `kerberos.auth_tgt` | The supplied principal/password can obtain a TGT (only runs when credentials are supplied). | Critical |
| `kerberos.auth_tgs` | A TGS-REQ succeeds for the supplied target service (only runs when credentials and a target service are supplied). | Warning |

The HTML report surfaces the most common misconfigurations with a direct remediation hint: no SRV records (publish `_kerberos._tcp.REALM. SRV …`), an SRV target with no A/AAAA, port 88 unreachable (open TCP+UDP 88), clock skew above the maximum (run ntpd/chrony), weak-enctype-only realms (switch to `aes256-cts-hmac-sha1-96`), the wrong realm in the reply, and AS-REP roasting exposure (enable `requires_preauth`).

## Options

| Option | Meaning | Default |
|---|---|---|
| Kerberos realm | DNS domain advertising the realm (auto-filled from the service scope; the realm name is derived in uppercase). Required. | *(auto)* |
| Principal | Optional. Supply to run an authenticated round-trip; leave blank for anonymous probes only. | *(empty)* |
| Password | Optional, secret. Password for the principal above; used once per run and never stored. | *(empty)* |
| Service to request (TGS) | Optional. SPN requested via TGS-REQ once a TGT is acquired. Defaults to `krbtgt` (realm self-test). | *(empty)* |
| Per-probe timeout (seconds) | Timeout for each probe. | `5` |
| Require strong enctypes | When enabled, realms advertising only DES/RC4 are flagged as Critical. | `true` |
| Max tolerated clock skew (seconds) | Default Kerberos tolerance is 300 s; tighter values surface drift earlier. | `300` |

## In happyDomain

Enable the Kerberos checker from the **Checks** tab of a Kerberos service. The realm domain is filled in automatically; supply a principal and password only if you want the authenticated TGT/TGS round-trip to run. See {{< relref "/pages/checks" >}} for the full workflow.
