---
date: 2026-06-10T12:00:00+02:00
title: Provider features
author: nemunaire
weight: 700
description: "Compare what each DNS provider supports with the provider feature matrix"
---

Not every DNS provider supports the same set of capabilities. Some can list the domains in your account automatically, others handle only the most common record types, and a few support more specialised records such as `CAA` or `TLSA`. The **Supported providers** page presents this information as a single comparison table so you can pick the right provider, or understand why a given [service]({{% relref "services" %}}) is unavailable for one of your domains.

## Reading the feature matrix

The page lists every provider happyDomain integrates with, one per row, with its logo and name. Each column corresponds to a capability, and the cell shows whether that provider supports it:

- a green check mark means the capability **is supported**;
- a red cross means it **is not supported**.

<!-- TODO: screenshot of the provider feature matrix -->

The table scrolls horizontally if it is wider than your screen, and the header row stays visible while you scroll, so you can always tell which capability each column refers to.

## The capabilities

The matrix compares the following capabilities:

| Column | Meaning |
|--------|---------|
| **Supported providers** | The provider can automatically list the domains in your account, so you can import them without typing each name. When unsupported, you add domains manually. |
| **Common types** | The provider supports the everyday record types (such as `A`, `AAAA`, `MX`, `TXT`, `CNAME`). This is what most domains need. |
| **CAA** | Support for `CAA` records, which declare the certificate authorities allowed to issue certificates for your domain. |
| **OPENPGPKEY** | Support for `OPENPGPKEY` records, used to publish OpenPGP public keys in DNS. |
| **PTR** | Support for `PTR` records, used mainly for reverse DNS (mapping an IP address back to a name). |
| **SRV** | Support for `SRV` records, which advertise the location of a service (port and host) for protocols that rely on them. |
| **SSHFP** | Support for `SSHFP` records, which publish SSH host key fingerprints in DNS. |
| **TLSA** | Support for `TLSA` records, used by DANE to bind a certificate to a name. |

{{% notice style="info" title="Why this matters" icon="circle-info" %}}
When you add a [service]({{% relref "services" %}}) to a subdomain, happyDomain only offers the service types your provider can actually publish. If a service you expect is greyed out, the feature matrix is the place to confirm whether the underlying record type is supported by that provider.
{{% /notice %}}

## Choosing a provider

If you are still deciding where to host your DNS, use this page to make sure the provider you have in mind covers the record types you need. A provider that supports the **Common types** is enough for most websites and email setups; the specialised columns matter only if you rely on the corresponding features (DANE, reverse DNS, SSH fingerprints, and so on).

Once you have picked a provider, head over to [add it]({{% relref "provider-new-choice" %}}) to happyDomain.
