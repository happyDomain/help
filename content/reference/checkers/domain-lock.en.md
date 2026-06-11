---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Domain transfer lock
description: "Verifies that a domain carries the expected EPP lock statuses that protect it against unauthorized transfers or changes."
weight: 20
---

The **Domain transfer lock** checker verifies that a domain carries the EPP status codes that protect it against unauthorized transfers, updates or deletions. A locked domain (for example one bearing `clientTransferProhibited`) cannot be transferred away without first removing the lock, which is a key defence against domain hijacking.

This is a **domain-level** checker: the status codes are read from the registry through a WHOIS/RDAP lookup, not from the zone's DNS records.

## What it checks

A single rule, `domain_lock_check`, compares the EPP status codes reported by the registry against the list of statuses you require.

| Status | Condition |
|--------|-----------|
| **OK** | Every required status is present on the domain |
| **Critical** | One or more required statuses are missing (the missing codes are listed) |
| **Unknown** | No required status is configured (nothing to check) |
| **Error** | The WHOIS/RDAP lookup failed |

Comparison is tolerant of formatting: spaces, dashes and underscores are ignored and case does not matter, so `clientTransferProhibited`, `client-transfer-prohibited` and `client transfer prohibited` are all treated as equal.

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Required lock statuses | Comma-separated list of EPP status codes that must be present on the domain (for example `clientTransferProhibited`, `clientUpdateProhibited`, `clientDeleteProhibited`). At least one code must be supplied. | `clientTransferProhibited` |

## In happyDomain

Enable this checker from the domain's **Checks** view; see {{< relref "/pages/checks" >}} for how to configure and schedule checks. The domain name is filled in automatically.

This checker pairs naturally with {{< relref "/reference/checkers/domain-expiry" >}} and {{< relref "/reference/checkers/domain-contact" >}} to keep the registration of a domain under control.
