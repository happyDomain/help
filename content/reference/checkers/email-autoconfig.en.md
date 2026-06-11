---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Mail autoconfiguration
description: "Verify that a domain publishes discoverable mail-client configuration through Thunderbird autoconfig, Microsoft Autodiscover and RFC 6186 SRV records."
weight: 220
---

The **Mail Autoconfiguration** checker verifies that mail clients can automatically discover how to connect to your mail servers. When autoconfiguration is published correctly, a user only types their email address and password, and their client (Thunderbird, Outlook, mobile mail apps…) finds the right IMAP/POP and SMTP hosts, ports and security settings on its own.

This checker is **service-level**: it applies to the mail-autoconfiguration and RFC 6186 services of a domain. For each run it probes the discovery mechanisms used by real-world clients:

- **Thunderbird autoconfig** (Bucksch draft): `https://autoconfig.<domain>/mail/config-v1.1.xml` as the primary URL, with the `https://<domain>/.well-known/autoconfig/...` apex fallback, an optional plain-HTTP variant, the Mozilla ISPDB fallback, and MX-parent fallbacks.
- **Microsoft Autodiscover** (POX): `https://autodiscover.<domain>/autodiscover/autodiscover.xml`.
- **RFC 6186 SRV records**: `_imaps`, `_imap`, `_pop3s`, `_pop3`, `_submissions`, `_submission`, `_autodiscover`.
- **MX resolution**, for context and MX-based discovery.

It parses every response, cross-checks the servers advertised by the different sources, and produces an HTML report with paste-ready remediation snippets for the most common failure modes.

## What it checks

| Rule | What it verifies | Severity on failure |
|---|---|---|
| `autoconfig_presence` | At least one autoconfiguration discovery method answers for the domain. | Critical |
| `autoconfig_preferred_endpoint` | The primary URL `https://autoconfig.<domain>/mail/config-v1.1.xml` is reachable and serves a valid clientConfig. | Warning |
| `autoconfig_tls` | Autoconfig endpoints are served over HTTPS with a valid TLS certificate. | Critical |
| `autoconfig_server_encryption` | Servers advertised by autoconfig use SSL or STARTTLS and a non-cleartext authentication method. | Critical |
| `autoconfig_consistency` | Hostnames and ports reported by autoconfig, Autodiscover and SRV records agree with each other. | Warning |
| `autoconfig_srv_records` | RFC 6186 SRV records complement the autoconfig XML. | Warning |
| `autoconfig_autodiscover` | Whether Microsoft Autodiscover (POX) responds on the domain. | Warning |

When a check fails, the report's "Fix this first" section provides ready-to-copy snippets: a sample `config-v1.1.xml` and its canonical URLs when nothing is published, a nudge to add the `autoconfig.` subdomain when only `.well-known` answers, an HTTPS redirect hint, a certificate-coverage hint, a port cheat-sheet for plaintext servers (SSL 993/465, STARTTLS 143/587), and a ready-to-paste SRV zone excerpt.

## Options

### Per user

| Option | Meaning | Default |
|---|---|---|
| Local-part used in probes (`probeEmail`) | Local part sent in the autoconfig URL query string (before `@`). Most servers ignore it. | `test` |
| HTTP timeout (`httpTimeout`) | Per-request timeout, in seconds, when probing endpoints. | 8 |
| Try Mozilla ISPDB fallback (`tryISPDB`) | When the domain publishes no autoconfig file, also query Mozilla's public Thunderbird ISPDB. | true |
| Allow plain-HTTP fallback probe (`tryHTTPAutoconfig`) | Also probe the plain-HTTP variant of `autoconfig.<domain>` (optional in the draft); useful to spot HTTP-only providers. | false |
| Probe Microsoft Autodiscover (`tryAutodiscoverPost`) | Probe the Exchange/Outlook Autodiscover endpoints. Disable to check only the Thunderbird flow. | true |

### Admin

| Option | Meaning | Default |
|---|---|---|
| Mozilla ISPDB base URL (`ispdbURL`) | Base URL of Mozilla's autoconfig fallback database. | `https://autoconfig.thunderbird.net/v1.1/` |
| User-Agent used in probes (`userAgent`) | User-Agent announced in every probe request. | `happyDomain-autoconfig/1.0 (+https://happydomain.org)` |

## In happyDomain

Enable this checker from the **Checks** tab of the relevant mail-autoconfiguration service. See {{< relref "/pages/checks" >}} for scheduling and reading checks.

This checker is the natural companion to a full mail setup: see {{< relref "/reference/services/email" >}} for the MX, SPF, DKIM and DMARC services that govern how mail is delivered and authenticated.
