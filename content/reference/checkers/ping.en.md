---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Ping
description: "Sends ICMP probes to every address of a service and reports reachability, average round-trip time, packet loss, and IPv6 coverage."
weight: 190
---

The **Ping (ICMP)** checker measures basic network reachability for the addresses behind a service. It sends a small number of ICMP echo requests to each `A`/`AAAA` address and reports whether the target replied, its average round-trip time (RTT), the observed packet-loss ratio, and whether at least one IPv6 address answered.

**Scope:** service-level. It attaches to services of type `abstract.Server` (a subdomain that publishes `A`/`AAAA` records) and is configured from that service's **Checks** tab.

## What it checks

| Rule | Verifies | Warn / Critical conditions |
|------|----------|----------------------------|
| `ping.reachable` | Every target replied to at least one ICMP probe. | Critical when a target never replies. |
| `ping.rtt` | Average round-trip time stays within thresholds. | Warning above the warning RTT, Critical above the critical RTT. |
| `ping.ipv6_reachable` | At least one IPv6 target replied to an ICMP probe. | Warning when no IPv6 address answers. |
| `ping.packet_loss` | Packet-loss ratio stays within thresholds. | Warning above the warning loss ratio, Critical above the critical loss ratio. |

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Warning RTT threshold (ms) (`warningRTT`) | Average RTT above which the check warns. | 100 |
| Critical RTT threshold (ms) (`criticalRTT`) | Average RTT above which the check is critical. | 500 |
| Warning packet loss threshold (%) (`warningPacketLoss`) | Packet-loss ratio above which the check warns. | 10 |
| Critical packet loss threshold (%) (`criticalPacketLoss`) | Packet-loss ratio above which the check is critical. | 50 |
| Number of pings to send (`count`) | ICMP echo requests sent per target. | 5 |

## In happyDomain

This is a service-level checker: configure it from the **Checks** tab of the *Server* service on the relevant subdomain. Its short default interval makes it well suited to lightweight, frequent reachability monitoring. For the general workflow of configuring and reading checks, see {{< relref "/pages/checks" >}}.
