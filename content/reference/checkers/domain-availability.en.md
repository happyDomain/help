---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Domain availability
description: "Notifies you when a watched domain name becomes available for registration."
weight: 40
---

The **Domain availability** checker watches a domain name you do *not* own and notifies you the moment it becomes available for registration. It is the counterpart of {{< relref "/reference/checkers/domain-expiry" >}}: instead of protecting a domain you hold, it lets you grab one as soon as it lapses.

This is a **domain-level** checker: registration status is determined from the registry through a WHOIS/RDAP lookup. A domain is considered available when the registry reports that it does not exist.

## What it checks

A single rule, `domain_availability_check`, reports whether the watched domain is still registered or has become free.

| Status | Condition |
|--------|-----------|
| **Critical** | The domain is now available for registration |
| **OK** | The domain is still registered (the registrar and expiry date are reported when known) |
| **Error** | The availability lookup failed |

{{% notice style="info" title="Why available is reported as Critical" %}}
The status is intentionally inverted compared with the usual convention. Reporting *Critical* when the domain becomes available makes the registered → available transition cross the notification threshold, so you are alerted exactly once when the domain frees up.
{{% /notice %}}

## Options

This checker has no user-tunable options. The watched domain name is supplied automatically.

## In happyDomain

Unlike the other domain-level checkers, **Domain availability** is not scheduled on the domains you manage. It is driven by the dedicated availability-watch list. See {{< relref "/pages/domain-availability" >}} for how to add a domain to watch, and {{< relref "/pages/checks" >}} for the general checks workflow.
