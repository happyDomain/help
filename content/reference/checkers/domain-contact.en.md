---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Domain contacts
description: "Verifies that the registered domain contacts (registrant, admin, tech) match the values you expect, with detection of privacy-protected records."
weight: 30
---

The **Domain contacts** checker compares the contact information registered for a domain (registrant, administrative and technical contacts) against the values you expect. An unexpected change to a contact, especially the registrant, can be an early sign of a hijack or of an administrative mistake.

This is a **domain-level** checker: the contact data is read from the registry through a WHOIS/RDAP lookup. Many registries redact contact fields for privacy; this checker detects redaction and reports it rather than treating it as a mismatch.

## What it checks

A single rule, `domain_contact_check`, evaluates each contact role you selected and emits one result per role.

| Status | Condition |
|--------|-----------|
| **OK** | The role's contact matches every expected value provided |
| **Warning** | A field does not match the expected value, or the role is missing from the record |
| **Info** | The contact is privacy-protected (redacted) so it cannot be compared |
| **Unknown** | No expected value is configured, or no role is selected (nothing to check) |
| **Error** | The WHOIS/RDAP lookup failed |

Comparisons are case-insensitive and exact. Redaction is detected from common markers in the contact fields such as *redacted*, *privacy*, *withheld*, *not disclosed*, *data protected*, *contact privacy* and *whoisguard*.

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Expected registrant name | If set, the configured roles must report this exact name (case-insensitive). | *(empty)* |
| Expected organization | If set, the configured roles must report this exact organization (case-insensitive). | *(empty)* |
| Expected email | If set, the configured roles must report this exact email (case-insensitive). | *(empty)* |
| Contact roles to check | Comma-separated list of roles among `registrant`, `admin`, `tech`. | `registrant` |

{{% notice style="info" title="At least one expected value is required" %}}
If none of the expected name, organization or email is set, the checker has nothing to compare and reports an *Unknown* status. Set at least one expected value for the check to be meaningful.
{{% /notice %}}

## In happyDomain

Enable this checker from the domain's **Checks** view; see {{< relref "/pages/checks" >}} for how to configure and schedule checks. The domain name is filled in automatically.

This checker complements {{< relref "/reference/checkers/domain-expiry" >}} and {{< relref "/reference/checkers/domain-lock" >}}, which together keep the registration of a domain under watch.
