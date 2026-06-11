---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Blacklist (DNSBL)
description: "Checks whether a domain is currently listed on widely-used reputation systems: DNS-based domain blocklists, downloaded phishing/malware feeds, and threat-intelligence APIs."
weight: 320
---

The **Blacklist & reputation** checker flags whether a domain is currently listed on widely-used reputation systems. It queries a range of sources in parallel and produces a diagnosis-first HTML report whose "Action required" section lists the most impactful listings with a per-operator removal procedure.

**Scope:** domain-level. The check targets the domain name itself, independent of its DNS records, and is enabled from the domain's checks view.

## What it checks

Each configured source contributes its own per-source rule; a listing on any of them raises the domain's status. The sources fall into three families:

- **DNS-based domain blocklists (DBL/RHSBL)** — queried over DNS, no API key required: Spamhaus DBL, SURBL multi, URIBL multi, NordSpam DBL, SpamEatingMonkey Fresh, Tiopan DBL, SORBS RHSBL, plus any extra DNSBL zones an administrator adds.
- **Downloaded phishing/malware feeds** — fetched and cached in memory: OpenPhish public feed, PhishTank, Botvrij.eu, Disconnect.me, OISD, and a Quad9 secure-DNS comparison.
- **Threat-intelligence HTTPS lookups** — requiring an API key configured by an administrator: Google Safe Browsing, abuse.ch URLhaus / ThreatFox / MalwareBazaar, VirusTotal v3, AlienVault OTX, Pulsedive, Criminal IP.

When a DNSBL query is refused (many public resolvers are blocked by DBL operators) or an API quota is exhausted, the source is surfaced as a Warning so it does not pollute the OK status. A multi-vendor source such as VirusTotal reports Critical when at least one vendor flags the domain as malicious, and Warning when it is only suspicious.

## Options

The only domain-level option is the target itself:

| Option | Meaning | Default |
|--------|---------|---------|
| Domain name (`domain_name`) | The domain to look up. Auto-filled from the domain. | (from domain) |

All source selection and credentials are configured per source. Most sources are toggled on or off (and API keys supplied) at the **administrator** level; a subset of the downloaded-feed sources default to on and can be toggled by the user. See the source's own documentation for which sources need an API key and how to obtain one.

## In happyDomain

This is a domain-level checker: enable it from the domain's checks view. Listings often reflect a compromise; treat a positive result as a signal to audit the host and rotate credentials, then follow the per-operator delisting links shown in the report. For the general workflow of configuring and reading checks, see {{< relref "/pages/checks" >}}.
