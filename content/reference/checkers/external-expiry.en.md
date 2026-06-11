---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: External-reference expiry
description: "Retrieves the WHOIS/RDAP registration facts for the external domains your zone points to, so their expiry can be evaluated."
weight: 50
---

The **External-reference expiry** checker looks up the registration facts of the *external* domains your zone refers to. When a record points at a name you do not own (a `CNAME` to a third-party service, an `MX` to an external provider, an `NS` delegation…), the safety of your zone depends on that external domain staying registered. If it expires and is re-registered by someone else, the pointer can be hijacked.

This is a **zone-level** checker: it enumerates the external targets discovered in the zone and performs one WHOIS/RDAP lookup per distinct registrable domain. Targets sharing the same registrable domain reuse a single lookup, and lookups run with a bounded concurrency so a large zone does not overwhelm the registry.

## What it checks

This checker is primarily a **collector**. It gathers per-target WHOIS facts (registrable name, expiry date, creation date, registrar, status) and publishes them for the *dangling-reference* checker, which is where the actionable verdicts about expiration, redemption or recent re-registration are emitted.

Its own rule, `external_whois_collected`, reports only how the collection went:

| Status | Condition |
|--------|-----------|
| **OK** | WHOIS was collected for every external target |
| **Info** | Some lookups succeeded and some failed (a partial result), or no external target was reported at all |
| **Warning** | The WHOIS lookup failed for *all* external targets |
| **Error** | The external WHOIS observation could not be read |

{{% notice style="info" title="Verdicts live in the dangling-reference checker" %}}
This checker does not decide whether an external domain is dangerously close to expiry. It only retrieves the facts. The expiry, redemption and re-registration verdicts are surfaced by the companion dangling-reference checker, which consumes the facts collected here.
{{% /notice %}}

## Options

This checker has no user-tunable options. The list of external targets is supplied automatically from the zone's discovered references.

## In happyDomain

Enable this checker from the zone's **Checks** view; see {{< relref "/pages/checks" >}} for how to configure and schedule checks. The discovery of external targets is automatic.

For the registration of the domains you own directly, see {{< relref "/reference/checkers/domain-expiry" >}}.
