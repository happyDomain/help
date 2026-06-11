---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: SMTP
description: "Probes every MX target of a domain on port 25 the way an operator would with swaks: TCP connect, banner, EHLO, STARTTLS, mail-transaction and open-relay probes, reverse DNS and IPv6 coverage."
weight: 200
---

The **Inbound SMTP (MX posture)** checker exercises the *inbound* side of a domain's mail service. For every MX target of the zone it performs the live probes a human operator would run with `swaks` or `telnet … 25`: TCP connect, ESMTP banner and EHLO, STARTTLS negotiation, mail-transaction probes (null sender, postmaster, open-relay), reverse DNS / FCrDNS, extension inventory, and IPv4/IPv6 coverage. The result is an actionable HTML report.

**Scope:** service-level. It attaches to services of type `svcs.MXs` (the DNS-level MX record set) and is configured from that service's **Checks** tab.

The probe answers "can this domain *receive* mail correctly?". It does **not** test outbound deliverability (SPF/DKIM/DMARC alignment, spam scoring, blacklist status), which is the job of the {{< relref "/reference/checkers/happydeliver" >}} checker. Mail-transaction probes always stop at `RCPT` and emit `RSET`: no `DATA` is sent, so no mail is delivered.

## What it checks

| Rule | Verifies | Severity |
|------|----------|----------|
| `smtp.null_mx` | Reports whether the domain publishes a null MX (RFC 7505). | Info |
| `smtp.mx_present` | The domain publishes at least one MX record (or a null MX). | Critical |
| `smtp.mx_sanity` | Flags MX targets violating RFC 5321 § 5.1 (IP literals, CNAME chains, unresolved names). | Critical |
| `smtp.endpoint_reachable` | Every MX endpoint accepts a TCP connection on port 25. | Critical |
| `smtp.banner_sanity` | Every reachable endpoint emits a 220 SMTP greeting. | Critical |
| `smtp.ehlo_supported` | Every endpoint accepts EHLO. | Critical |
| `smtp.starttls_offered` | Every endpoint advertises the STARTTLS extension. | Critical |
| `smtp.starttls_handshake` | The STARTTLS handshake succeeds wherever advertised. | Critical |
| `smtp.auth_posture` | Flags endpoints advertising SMTP AUTH before STARTTLS (cleartext credentials). | Critical |
| `smtp.reverse_dns` | Every endpoint has a matching PTR record (FCrDNS). | Warning |
| `smtp.null_sender` | Endpoints accept the null sender `MAIL FROM:<>` (required for DSNs). | Critical |
| `smtp.postmaster` | Endpoints accept `RCPT TO:<postmaster@domain>` (RFC 5321 § 4.5.1). | Critical |
| `smtp.open_relay` | Flags endpoints that relay mail for recipients outside the tested domain. | Critical |
| `smtp.extension_posture` | Reports ESMTP extension posture (PIPELINING, 8BITMIME). | Info |
| `smtp.ipv6_reachable` | At least one MX endpoint is reachable over IPv6. | Info |
| `smtp.tls_quality` | Folds downstream TLS findings (chain, hostname, expiry) onto SMTP. | Critical |

Certificate posture itself is out of scope here: each MX target is published as a `tls.endpoint.v1` discovery entry (opportunistic STARTTLS), and the {{< relref "/reference/checkers/tls" >}} checker runs certificate analysis on the same connection. Its findings are folded back into the `smtp.tls_quality` rule and the HTML report.

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Domain (`domain`) | Domain to test. Auto-filled from the service. | (from service) |
| Per-endpoint timeout (seconds) (`timeout`) | Per-endpoint connection timeout. | 12 |
| EHLO hostname (`helo_name`) | Hostname announced in EHLO/HELO. Use a name that resolves and has a valid PTR. | `mx-checker.happydomain.org` |
| Probe null sender (`test_null_sender`) | Probe `MAIL FROM:<>` (RFC 5321 DSN acceptance). | true |
| Probe postmaster (`test_postmaster`) | Probe `RCPT TO:<postmaster@domain>` (RFC 5321 § 4.5.1). | true |
| Probe open-relay posture (`test_open_relay`) | Probe a recipient outside the tested domain to detect open relays. | true |
| Open-relay probe recipient (`test_probe_address`) | Mailbox (outside the tested domain) used for the open-relay probe. | `postmaster@example.com` |

## In happyDomain

This is a service-level checker: configure it from the **Checks** tab of the *E-Mail servers* (MX) service. To confirm that mail your domain *sends* lands in the inbox, pair it with the {{< relref "/reference/checkers/happydeliver" >}} checker. For the general workflow of configuring and reading checks, see {{< relref "/pages/checks" >}}.
