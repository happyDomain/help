---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Mail keys (DKIM / OpenPGP)
description: "Validate DNS-published OpenPGP keys (OPENPGPKEY) and S/MIME certificates (SMIMEA) used for end-to-end mail encryption, including their DNSSEC authentication."
weight: 230
---

The **Mail keys** checker (named *OPENPGPKEY & SMIMEA* in happyDomain) validates the cryptographic keys a domain publishes in DNS so that correspondents can encrypt mail to its users. It covers two DANE-style record types:

- **`OPENPGPKEY`** ([RFC 7929](https://www.rfc-editor.org/rfc/rfc7929)) — an individual user's OpenPGP public key, published under an owner-hashed name below `._openpgpkey.<zone>`.
- **`SMIMEA`** ([RFC 8162](https://www.rfc-editor.org/rfc/rfc8162)) — a user's S/MIME certificate, published under an owner-hashed name below `._smimecert.<zone>`.

This checker is **service-level**: it applies to the OpenPGP and S/MIME services of a subdomain and runs a comprehensive test suite, then renders an HTML report whose top block points to the fix for the most common failure scenarios.

{{% notice style="info" title="Publication and structure, not cryptographic trust" %}}
This checker validates DNS publication and the structure and metadata of the keys it finds. It does **not** cryptographically verify them: OpenPGP signatures (self-signatures, third-party certifications) are not verified, and S/MIME chains are not built or validated against any trust anchor (no CRL/OCSP). Authenticity of the records themselves is delegated to a validating resolver via the DNSSEC `AD` flag. Treat a green report as "the record is well-formed and DNSSEC-signed", not as "the key is trustworthy".
{{% /notice %}}

## What it checks

### DNS and DNSSEC

| Rule | What it verifies | Severity |
|---|---|---|
| `dns_query_failed` | The DNS lookup for the record succeeds. | Critical |
| `dns_no_record` | A record is published at the expected owner name. | Critical |
| `dns_record_mismatch` | The record returned by DNS matches the service-declared record. | Warning |
| `dnssec_not_validated` | The record is authenticated by DNSSEC (`AD` flag set). | Critical (Warning if DNSSEC not required) |
| `owner_hash_mismatch` | The owner-name first label equals `hex(sha256(username))[:28]`. | Critical |

### OpenPGP (`OPENPGPKEY`)

| Rule | What it verifies | Severity |
|---|---|---|
| `pgp_parse_error` | The record decodes as a valid OpenPGP key. | Critical |
| `pgp_primary_revoked` | The primary key carries no revocation signature. | Critical |
| `pgp_primary_expired` | The primary key has not passed its self-signature expiry. | Critical |
| `pgp_primary_expiring_soon` | The primary key does not expire within the configured window. | Warning |
| `pgp_weak_algorithm` | No legacy algorithm (DSA/ElGamal) is used. | Warning |
| `pgp_weak_key_size` | RSA keys meet the minimum 2048-bit size (3072+ preferred). | Critical |
| `pgp_no_encryption_subkey` | At least one active key advertises encryption capability. | Critical |
| `pgp_no_identity` | The key carries at least one self-signed User ID. | Warning |
| `pgp_uid_mismatch` | At least one UID references `<username@...>`. | Info |
| `pgp_multiple_entities` | The record carries a single OpenPGP entity (RFC 7929). | Warning |
| `pgp_record_too_large` | The record stays below 4 KiB to fit typical UDP answers. | Warning |

### S/MIME (`SMIMEA`)

| Rule | What it verifies | Severity |
|---|---|---|
| `smimea_bad_usage` | The usage field is 0, 1, 2 or 3. | Critical |
| `smimea_bad_selector` | The selector field is 0 (Cert) or 1 (SPKI). | Critical |
| `smimea_bad_match_type` | The matching type is 0 (Full), 1 (SHA-256) or 2 (SHA-512). | Critical |
| `smimea_cert_parse_error` | The record decodes as a valid X.509 certificate or SPKI. | Critical |
| `smimea_cert_not_yet_valid` | The certificate's `NotBefore` is in the past. | Critical |
| `smimea_cert_expired` | The certificate's `NotAfter` is in the future. | Critical |
| `smimea_cert_expiring_soon` | The certificate does not expire within the configured window. | Warning |
| `smimea_no_email_protection_eku` | The certificate advertises the `emailProtection` EKU. | Critical (Warning if not required) |
| `smimea_missing_key_usage` | The certificate carries `digitalSignature` and/or `keyEncipherment` key usage. | Warning |
| `smimea_weak_signature_algorithm` | The certificate is not signed with a deprecated algorithm (MD2/MD5/SHA-1). | Critical |
| `smimea_weak_key_size` | RSA keys meet the minimum 2048-bit size (3072+ preferred). | Critical |
| `smimea_self_signed` | Flags self-signed certificates paired with PKIX-EE (usage 1). | Info |
| `smimea_email_mismatch` | At least one email SAN begins with `<username>@`. | Info |
| `smimea_hash_only` | Notes that matching types 1/2 transport only a digest, preventing certificate inspection. | Info |

## Options

| Option | Meaning | Default |
|---|---|---|
| DNS resolver (`resolver`) | Validating resolver to query (comma-separated list accepted). Empty uses the system resolver. | (system) |
| `certExpiryWarnDays` | Window, in days, for the `expiring_soon` warnings (PGP and S/MIME). | 30 |
| `requireDNSSEC` | When false, a missing `AD` flag is a Warning instead of Critical. | true |
| `requireEmailProtection` | When false, a missing `emailProtection` EKU is a Warning instead of Critical. | true |

The domain origin, subdomain, service and service type are auto-filled by happyDomain.

{{% notice style="info" title="Query a validating resolver" %}}
Because record authenticity is delegated to DNSSEC, run this checker against a resolver you trust to perform DNSSEC validation, so the `AD` flag reflects a real validation.
{{% /notice %}}

## In happyDomain

Enable this checker from the **Checks** tab of the relevant OpenPGP or S/MIME service. See {{< relref "/pages/checks" >}} for the general workflow.

These records share their security model with DNSSEC: to confirm your zone's signing chain is itself sound, see {{< relref "/reference/checkers/dnssec" >}}. For the surrounding mail configuration, see {{< relref "/reference/services/email" >}}.
