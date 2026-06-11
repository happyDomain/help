---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: STUN / TURN
description: "Probes STUN and TURN servers end-to-end: discovery, reachability, TLS/DTLS, STUN binding and authenticated TURN relay."
weight: 310
---

The **STUN / TURN** checker probes STUN and TURN servers end-to-end. STUN and TURN are the NAT-traversal servers that real-time applications (WebRTC, voice and video) rely on to establish peer-to-peer media: STUN lets a host discover its public reflexive address, while TURN relays media when a direct path cannot be opened.

This is a **service-level** checker. It runs SRV discovery (or uses an explicit URI), checks TCP/UDP reachability and the TLS/DTLS handshake, issues a STUN binding request, verifies that the TURN server requires authentication, performs an authenticated TURN `Allocate`, and finally exercises the relay path with a `CreatePermission + Send` round-trip.

## What it checks

| Rule | What it verifies | Severity |
|------|------------------|----------|
| `stun_turn.discovery` | At least one STUN/TURN endpoint could be discovered (explicit URI or SRV lookup). | Critical |
| `stun_turn.srv_stun` | At least one STUN endpoint is available via SRV (`_stun` / `_stuns`) or explicit URI. | Warning |
| `stun_turn.srv_turn` | At least one TURN endpoint is available via SRV (`_turn` / `_turns`) or explicit URI. | Critical |
| `stun_turn.dial` | Every discovered endpoint accepts a connection (TCP/TLS handshake or UDP socket). | Critical |
| `stun_turn.tls_transport` | At least one TLS/DTLS transport (`stuns` / `turns`) succeeds when present. | Critical |
| `stun_turn.ipv6_coverage` | At least one STUN/TURN hostname resolves to an IPv6 address. | Warning |
| `stun_turn.stun_binding` | The STUN Binding request receives a XOR-MAPPED-ADDRESS reply. | Critical |
| `stun_turn.reflexive_public` | Flags endpoints returning a private/loopback reflexive address (server unaware of its public IP). | Critical |
| `stun_turn.stun_latency` | Compares the STUN Binding RTT against the warning/critical thresholds. | Critical |
| `stun_turn.turn_open_relay` | The TURN server requires authentication (challenges an unauthenticated `Allocate` with 401). | Critical |
| `stun_turn.turn_auth` | The supplied TURN credentials (or REST shared secret) yield a successful `Allocate`. | Critical |
| `stun_turn.relay_public` | Flags TURN servers whose allocated relay address is private/loopback (missing public relay IP). | Critical |
| `stun_turn.relay_echo` | The TURN relay path can carry traffic to the configured probe peer (`CreatePermission + Send`). | Warning |

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Zone | Zone used for SRV-based discovery (`_stun._udp` / `_turn._udp` / `_turns._tcp`) when no explicit URI is given. Filled in automatically. | (auto-filled) |
| Server URI | Explicit STUN/TURN URI (RFC 7064/7065). Overrides SRV-based discovery. | — |
| Mode | `auto` probes both STUN and TURN; `stun` skips TURN allocation tests; `turn` requires TURN allocation. | `auto` |
| TURN username | Username for long-term TURN credentials. | — |
| TURN password | Password for long-term TURN credentials (secret). | — |
| REST API shared secret | Shared secret to derive ephemeral credentials (draft-uberti-rtcweb-turn-rest); takes precedence over username/password (secret). | — |
| Realm | Optional explicit TURN realm. | — |
| Transports | Comma-separated transports to test among `udp`, `tcp`, `tls`, `dtls`. | `udp,tcp,tls` |
| Relay echo target | `host:port` used to validate the relay path; a `CreatePermission + Send` is issued, no payload data is exchanged. | `1.1.1.1:53` |
| Also test ChannelBind | Additionally exercise ChannelBind through the relay connection. | `false` |
| RTT warning threshold (ms) | STUN Binding round-trip time above which a warning is raised. | 200 |
| RTT critical threshold (ms) | STUN Binding round-trip time above which a critical alert is raised. | 1000 |
| Per-probe timeout (s) | Time budget for each individual probe. | 5 |

{{% notice style="info" title="Credentials are needed for the TURN tests" %}}
The authentication, relay-public and relay-echo rules only run when valid TURN credentials are provided — either a username/password pair or a REST API shared secret. Without them, the checker still validates discovery, reachability, TLS and STUN binding, but cannot exercise the TURN relay path.
{{% /notice %}}

## In happyDomain

Enable this checker from the **Checks** tab of the relevant service; see {{< relref "/pages/checks" >}} for how to configure and schedule checks. The zone is filled in automatically; supply a server URI and TURN credentials as needed for your deployment.
