---
date: 2026-06-10T12:00:00+02:00
title: Domain availability & lookups
author: nemunaire
weight: 1800
description: "Check whether a domain is available for registration, and inspect any domain with the WHOIS and DNS resolver tools"
---

happyDomain bundles a few diagnostic tools that work on any domain name, whether or not you manage it in happyDomain. They let you check if a name is available for registration, look up its registration details (WHOIS), and query its DNS records directly (resolver).

## Availability checker

The **Availability** page lets you keep an eye on one or more domain names you would like to register, and be told as soon as one becomes available.

<!-- TODO: screenshot of the availability page -->

### Watching a domain

To start watching a name:

1. Type the domain you are interested in (for example `mydomain.example`) in the input at the top of the page.
2. Click **Add**.

The name is added to your watch list and an immediate check is launched in the background, so you do not have to wait for the next automatic check to see its current state.

### Reading the status

Each watched domain shows a badge reflecting the result of the last check:

| Badge | Meaning |
|-------|---------|
| **Available** | The name appears to be free and can probably be registered. |
| **Registered** | The name is already taken. |
| _(error message)_ | The check could not be completed (for example the extension is unsupported, or the registry could not be reached). |
| **Never checked** | No result is available yet. |

The time of the last check is displayed next to the status.

### Rechecking and removing

- Click **Check now** next to a domain to trigger a fresh check immediately. A spinner is shown while the check runs in the background, and the status updates automatically once it finishes.
- Click the trash icon to stop watching a domain. A confirmation is asked before removal.

{{% notice style="tip" title="Availability is a best effort" icon="lightbulb" %}}
The availability result is only an indication. Some extensions cannot be checked reliably, and a name that appears free may still be reserved or blocked at registration time. Always confirm with a registrar before counting on a name.
{{% /notice %}}


## WHOIS lookup

The **WHOIS** tool shows the public registration information of a domain: its registrar, important dates and current status.

<!-- TODO: screenshot of the WHOIS lookup page -->

Enter a domain name and run the lookup. happyDomain then displays a clear summary of the collected information:

- **Status**: the registration statuses returned for the domain (for example `active`, `clientTransferProhibited`, a `hold` state, etc.), shown as colour-coded badges.
- **Creation date** and **Expiration date**: when the domain was first registered and when its registration is due to expire. The expiration is accompanied by a progress bar and a countdown, which turns orange then red as the date approaches.
- **Registrar**: the company through which the domain is registered, with a link to its website when available.
- **Nameservers**: the authoritative nameservers declared for the domain.

If the name is not registered, a message tells you the domain was not found. If the lookup itself fails, the error is shown.

{{% notice style="info" title="WHOIS and the availability checker" icon="circle-info" %}}
The availability checker relies on the same underlying registration data as the WHOIS lookup. If you want the full registration picture for a name, use the WHOIS tool; if you simply want to know whether a name is free (and be notified later), add it to your availability watch list.
{{% /notice %}}


## DNS resolver

The **DNS resolver** lets you query the live DNS of any domain, exactly as it is published on the Internet right now. This is handy to confirm that a change has propagated, or to inspect a domain you do not manage.

<!-- TODO: screenshot of the resolver page -->

Enter a domain name and run the query. By default, every record type is requested (`ANY`). Open the **Advanced** options to refine the query:

- **Field (record type)**: restrict the query to a single record type (`A`, `AAAA`, `MX`, `TXT`, etc.).
- **Resolver**: choose which DNS resolver to send the query to. Several well-known public resolvers are offered, and you can pick **Custom** to enter the address of any resolver you want to test against.
- **Show DNSSEC records**: include the DNSSEC-related records (`RRSIG`, `NSEC`, `NSEC3`) in the results, which are hidden by default.

If you manage domains in happyDomain, their names are suggested as you type the domain to query.

Results are grouped by record type, and each entry shows its fields together with its TTL. When the domain exists but has no record of the requested type, a message says so explicitly.

{{% notice style="tip" title="Resolver vs. your zone in happyDomain" icon="lightbulb" %}}
The resolver shows what is **currently published** by the authoritative servers, which may differ from a draft zone you are editing in happyDomain until you [publish your changes]({{% relref "publish-changes" %}}).
{{% /notice %}}
