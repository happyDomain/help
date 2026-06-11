---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Domain expiry
description: "Warns when a domain name is approaching its registration expiry date."
weight: 10
---

The **Domain expiry** checker watches the registration expiry date of a domain name and warns you before it lapses. Letting a domain expire can mean losing it to another registrant, so this is one of the most important domain-level checks.

This is a **domain-level** checker: it concerns the domain registration itself, not its DNS records. The expiry date is obtained from the registry through a WHOIS/RDAP lookup, together with the registrar name.

## What it checks

A single rule, `domain_expiry_check`, compares the number of days remaining until expiry against two thresholds and reports the corresponding status.

| Status | Condition |
|--------|-----------|
| **Critical** | Days remaining ≤ critical threshold |
| **Warning** | Days remaining ≤ warning threshold (but above critical) |
| **OK** | Days remaining above the warning threshold |
| **Error** | The WHOIS/RDAP lookup failed or no expiry date is available |

The message always reports how many days remain until expiry, regardless of status.

The checker also exposes a metric, `domain_expiry_days_remaining`, labelled with the registrar, so the time left can be tracked over time.

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Warning threshold (days) | Days before expiry at which a warning is raised. Must be positive. | 30 |
| Critical threshold (days) | Days before expiry at which a critical alert is raised. Must be positive. | 7 |

{{% notice style="info" title="Critical must be lower than warning" %}}
The critical threshold must be strictly smaller than the warning threshold. happyDomain rejects a configuration where `critical_days` is greater than or equal to `warning_days`.
{{% /notice %}}

## In happyDomain

Enable this checker from the domain's **Checks** view; see {{< relref "/pages/checks" >}} for how to configure and schedule checks. The domain name is filled in automatically.

For the inverse situation, watching a domain you do *not* yet own so you can register it once it lapses, see the {{< relref "/reference/checkers/domain-availability" >}} checker and {{< relref "/pages/domain-availability" >}}. Related domain-level checkers include {{< relref "/reference/checkers/domain-lock" >}} and {{< relref "/reference/checkers/domain-contact" >}}.
