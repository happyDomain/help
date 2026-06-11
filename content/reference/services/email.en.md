---
date: 2021-01-12T21:38:49+02:00
author: Frederic
title: E-Mail
description: "Declare the mail servers of a zone (MX records) and the related anti-spoofing services (SPF, DKIM, DMARC) that govern how mail is sent and received."
aliases:
    services/email
---

The **E-Mail** service lets you declare the mail servers responsible for a zone, and points you to the companion services that control how mail is sent and authenticated for that domain.

In happyDomain, this service is named **E-Mail servers**: its single job is to publish the `MX` records that tell the rest of the Internet *where* to deliver mail addressed to your domain. Everything that protects *outgoing* mail (proving that a message really comes from you) is handled by separate, dedicated services described further down.

## Receiving mail: MX records

An `MX` (Mail eXchanger) record names a host that accepts mail for the domain. A zone usually publishes several of them so that delivery keeps working if one server is unavailable.

Each entry has two parts:

- **Priority** — a number that orders the servers. Sending servers try the **lowest** number first; higher numbers are used only as fallbacks. Two records sharing the same priority are treated as equivalent and load-balanced.
- **Mail server** — the hostname of the machine that receives the mail (for example `mx1.example.com.`). This host must itself resolve to an address; it should not be a bare IP.

A common setup looks like this:

| Priority | Mail server |
|---|---|
| 10 | `mx1.example.com.` |
| 20 | `mx2.example.com.` |

Here `mx1` is tried first, and `mx2` is the backup.

{{% notice style="info" title="One E-Mail service per subdomain" %}}
The E-Mail servers service is *single*: a given subdomain can hold only one such service, which gathers all of its `MX` records together. To declare mail servers for both the apex (`example.com`) and a subdomain (`mail.example.com`), add the service to each of them separately.
{{% /notice %}}

## Sending mail: authentication and anti-spoofing

Publishing `MX` records is enough to *receive* mail, but it says nothing about which servers are allowed to *send* mail on your behalf. Without that, anyone can forge messages using your domain. happyDomain offers several dedicated services, each managing its own DNS records, to establish that posture:

- **SPF** (*Sender Policy Framework*) — a `TXT` record, usually at the zone apex, that lists the servers authorized to send mail for the domain. Receivers compare the sending server against this list.
- **DKIM** (*DomainKeys Identified Mail*) — publishes the public half of a signing key as a `TXT` record under a `._domainkey` selector. Your mail servers sign outgoing messages, and receivers verify the signature against this published key.
- **DMARC** (*Domain-based Message Authentication, Reporting and Conformance*) — a `TXT` record at `_dmarc` that tells receivers what to do with messages failing SPF or DKIM checks (let them through, quarantine, or reject), and where to send aggregate reports.

Two further services cover transport security and reporting:

- **MTA-STS** — declares that mail to your domain must be delivered over a secured (TLS) connection.
- **TLS-RPT** — collects reports about TLS delivery problems encountered by sending servers.

These services are independent of the E-Mail servers service. You can add only the ones you need, but a complete and well-protected mail configuration typically combines `MX`, SPF, DKIM and DMARC.

## In the zone editor

To configure mail for a subdomain in the happyDomain zone editor:

1. Add the **E-Mail servers** service to the subdomain and fill in one line per mail server, with its priority and hostname.
2. Add **SPF**, **DKIM** and **DMARC** as their own services on the relevant subdomain (SPF and DMARC usually at the apex). Each one presents a dedicated form rather than raw record text.

Because each of these is a distinct abstract service, you manage them separately even though they all work together to make mail for your domain both deliverable and trustworthy. See the {{< relref "/reference/services" >}} chapter for the full list of available services.
