---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: LDAP
description: "Probes a domain's LDAP directory end-to-end: SRV discovery, transport encryption (StartTLS / LDAPS), RootDSE introspection, anonymous exposure, plaintext-bind refusal, and optional authenticated bind."
weight: 250
---

The **LDAP** checker probes a domain's LDAP directory deployment from end to end. Starting from SRV discovery (`_ldap._tcp`, `_ldaps._tcp`), it tests reachability of every endpoint, confirms an encrypted channel is available (StartTLS per RFC 2830 or implicit TLS on port 636), introspects the RootDSE, looks for anonymous information disclosure, verifies the directory refuses cleartext binds, and — when credentials are supplied — performs an authenticated bind over TLS with an optional read test on a base DN.

This checker is **service-level**: it targets an *LDAP* service (`abstract.LDAP`) published on a subdomain and is configured from that service's own **Checks** tab. For each transport it probes `_ldap._tcp` (falling back to port 389) and `_ldaps._tcp` (falling back to port 636), testing each resolved A/AAAA address per IP family.

{{% notice style="info" title="TLS posture is folded in, not duplicated" %}}
The LDAP checker confirms only that a TLS session can be established, recording the negotiated version and cipher for context. Each probed endpoint is published as a `tls.endpoint.v1` discovery entry so the dedicated TLS checker can verify the certificate chain, hostname match and expiry. Those findings are folded back onto the LDAP service page through the `ldap.tls_quality` rule — a bad certificate on an LDAP endpoint shows up here, not only in a separate TLS view. See {{< relref "/reference/checkers/tls" >}}.
{{% /notice %}}

## What it checks

| Rule | What it verifies | Severity |
|---|---|---|
| `ldap.has_srv` | `_ldap._tcp` / `_ldaps._tcp` SRV records are published and resolvable. | Warning |
| `ldap.endpoint_reachable` | Every discovered LDAP endpoint accepts a TCP connection. | Critical |
| `ldap.has_encrypted_transport` | At least one reachable endpoint offers an encrypted channel (LDAPS or StartTLS). | Critical |
| `ldap.starttls_supported` | StartTLS is offered and the upgrade succeeds on every reachable plain LDAP endpoint. | Critical |
| `ldap.ldaps_handshake` | The direct TLS handshake succeeds on every LDAPS endpoint. | Critical |
| `ldap.starttls_on_ldaps` | Flags servers that needlessly advertise StartTLS on the implicit-TLS LDAPS port. | Info |
| `ldap.ipv6_reachable` | At least one endpoint is reachable over IPv6. | Info |
| `ldap.refuses_plain_bind` | The directory refuses authentication over a cleartext channel (`confidentialityRequired`, resultCode 13, per RFC 4513 §5.1.2). | Critical |
| `ldap.anonymous_search_blocked` | Flags directories that allow an anonymous `baseObject` search of the naming context (information disclosure). | Warning |
| `ldap.rootdse_readable` | The RootDSE is readable over TLS and advertises naming contexts. | Warning |
| `ldap.sasl_mechanisms` | Reviews `supportedSASLMechanisms`: presence of strong mechanisms (SCRAM-*, EXTERNAL, GSSAPI), absence of password-equivalent ones (PLAIN/LOGIN only). | Warning |
| `ldap.protocol_version` | Flags servers that still advertise the deprecated LDAPv2 protocol. | Warning |
| `ldap.bind_credentials` | The supplied bind credentials are accepted by the directory (only runs when `bind_dn` is set). | Critical |
| `ldap.base_dn_read` | The bound account can read the supplied base DN (only runs when `base_dn` is set and the bind succeeded). | Critical |
| `ldap.tls_quality` | Folds the downstream TLS checker findings (certificate chain, hostname match, expiry) onto the LDAP service. | Critical |

The HTML report leads with the most common failures and includes server-specific remediation (OpenLDAP `olcSecurity`, 389-ds `require_tls`…): no encrypted endpoint reachable, StartTLS missing on 389, a StartTLS handshake that fails, a cleartext bind accepted on 389, an LDAPS handshake that fails, anonymous search exposing the DIT, PLAIN/LOGIN-only SASL, lingering LDAPv2, and rejected bind credentials.

## Options

| Option | Meaning | Default |
|---|---|---|
| Domain | The directory's domain (auto-filled from the service scope). Required. | *(auto)* |
| Per-endpoint timeout (seconds) | Connection/probe timeout for each endpoint. | `10` |
| Bind DN | Optional. DN to bind as; used only when a bind password is also set, and only over a TLS-protected channel. | *(empty)* |
| Bind password | Optional, secret. The password is not persisted in the observation payload and is never sent over cleartext. | *(empty)* |
| Base DN (read test) | Optional. After a successful bind, a `baseObject` search on this DN confirms the account has read access. Falls back to an anonymous `baseObject` search when no bind DN is supplied. | *(empty)* |

## In happyDomain

Enable the LDAP checker from the **Checks** tab of an LDAP service. The domain is filled in automatically; supply a bind DN and password only if you want the authenticated bind and base-DN read tests to run. See {{< relref "/pages/checks" >}} for the full workflow, and {{< relref "/reference/checkers/tls" >}} for the certificate posture of the same endpoints.
