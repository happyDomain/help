---
date: 2021-01-12T21:38:49+02:00
author: Frederic
title: TXT
weight: 20
aliases:
    records/TXT
description: "The TXT record attaches free-form text to a DNS name. Learn what it is used for and how happyDomain edits it through the Text Record service."
---

A **TXT record** (defined in [RFC 1035](https://www.rfc-editor.org/rfc/rfc1035)) attaches one or more free-form text strings to a name in your zone. It carries no predefined meaning of its own, which makes it one of the most versatile record types: any application is free to define its own convention for the text it stores there.

## Common uses

Because a TXT record can hold arbitrary text, it has become the carrier for many widespread conventions:

- **Domain ownership and site verification** — a provider asks you to publish a token so it can confirm you control the domain.
- **SPF** — declares which servers are allowed to send e-mail for your domain.
- **DKIM** — publishes the public key used to sign your outgoing e-mail.
- **DMARC** — sets the policy applied when SPF or DKIM checks fail.
- Various other key or policy publications defined by different tools.

For most of these purposes, happyDomain offers dedicated, higher-level **services** (SPF, DKIM, DMARC, site verification, and more) that are easier and safer to use than a raw TXT record: they guide you with the right fields and validate the syntax for you. Browse them in the {{< relref "/reference/services" >}} chapter, in particular the e-mail-related services. Prefer those services whenever one matches your need.

{{% notice style="info" title="When a TXT record becomes a service" %}}
When happyDomain reads your zone, it recognises TXT records that follow a known convention (SPF, DKIM, DMARC, …) and presents them as their dedicated service rather than as a plain Text Record. Only TXT records without a recognised prefix or syntax are shown as a raw **Text Record**.
{{% /notice %}}

## Editing a TXT record in happyDomain

In the zone editor, a plain TXT record appears as a **Text Record** service. It is intentionally minimal: a single field holds the text content of the record.

To work with one:

1. Open the subdomain where the record should live (the apex of the zone, or any subdomain such as `www`).
2. Add or open the **Text Record** service.
3. Type the full text string in its only field.
4. Adjust the **TTL** if needed, then publish your changes to apply them.

The value you enter is stored verbatim. Editing it and publishing updates the corresponding TXT record on that subdomain.

{{% notice style="note" title="Long strings" %}}
A single text string inside a TXT record cannot exceed 255 bytes at the DNS protocol level. Longer values are automatically split into 255-byte chunks for you. You simply enter the complete string in happyDomain — no manual splitting is required.
{{% /notice %}}
