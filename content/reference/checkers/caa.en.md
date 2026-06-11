---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: CAA
description: "Cross-references the TLS certificates observed on a domain against its CAA issue/issuewild policy to detect unauthorized issuance."
weight: 150
---

The **CAA Compliance** checker verifies that the TLS certificates actually deployed on a domain were issued by a Certification Authority that the domain's `CAA` records authorize. A `CAA` (Certification Authority Authorization) record lets a domain declare which CAs may issue certificates for it; this checker confirms reality matches that policy.

This is a **service-level** checker: it runs on a `CAA` policy service. It performs **no network probes of its own** — it reads the parsed CAA policy from the service body and the TLS certificates observed by the {{< relref "/reference/checkers/dane" >}} / `checker-tls` family, then maps each observed issuer to its CAA identifier domain using the Common CA Database (CCADB).

## What it checks

A single rule, `caa_compliance`, loads the zone's CAA `issue` / `issuewild` policy, gathers every TLS probe observed on the target, resolves each certificate's issuer (by Authority Key Identifier, falling back to the issuer Distinguished Name) against the embedded CCADB "CAA Identifiers" mapping, and compares the result against the allow list.

| Outcome | Meaning | Status |
|---------|---------|--------|
| `caa_ok` | Every observed issuer is authorized by the CAA policy. | OK |
| `caa_no_tls` | No TLS probes related to this target have been published yet. | Unknown |
| `caa_not_authorized` | CCADB mapped an observed issuer to a domain the policy does not list. | Critical |
| `caa_issuance_disallowed` | The policy contains `CAA 0 issue ";"` (issuance explicitly forbidden) but a certificate was still observed. | Critical |
| `caa_issuer_unknown` | CCADB has no mapping for the observed issuer (AKI + DN). | Info |

{{% notice style="info" title="Eventual consistency with TLS probes" %}}
The `caa_no_tls` state is a normal steady state, not an error: until `checker-tls` has probed an endpoint on the target and published a certificate, the CAA checker has nothing to compare against and reports **Unknown**. Once probes arrive, the check resolves on its next run. The `caa_issuer_unknown` outcome means the CA simply isn't in the current CCADB snapshot; the fix is to file a CCADB update, not to change your zone.
{{% /notice %}}

The issuer-to-CAA-domain mapping comes from an embedded snapshot of CCADB's "CAA Identifiers (V2)" report. Refreshing it only requires re-embedding a newer CSV and recompiling — no code change and no network dependency at build time.

## Options

This checker has no user-tunable options. Both inputs are filled in automatically:

| Option | Meaning |
|--------|---------|
| Domain | The domain being checked (auto-filled, required). |
| Service | The CAA policy service body (auto-filled). |

## In happyDomain

Add the CAA policy service to the relevant subdomain (usually the apex), then enable this checker from that service's **Checks** tab. See {{< relref "/pages/checks" >}} for configuring and scheduling checks. For the comparison to produce a verdict, a TLS-probing checker such as {{< relref "/reference/checkers/dane" >}} (or the standalone `checker-tls`) must have observed certificates on the target.
