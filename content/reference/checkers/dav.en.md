---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: CalDAV / CardDAV
description: "Discovers and probes a domain's CalDAV and CardDAV servers: well-known/SRV discovery, HTTPS reachability, advertised DAV capabilities, and (with credentials) the authenticated principal, home-set, collections and REPORT flow."
weight: 240
---

The **CalDAV** and **CardDAV** checkers verify that a domain's calendar (RFC 4791) and contacts (RFC 6352) servers are discoverable, reachable, and correctly advertise the WebDAV extensions that clients rely on. They are two distinct checkers sharing the same logic: `caldav` attaches to a *CalDAV* service (`abstract.CalDAV`), `carddav` to a *CardDAV* service (`abstract.CardDAV`). The only behavioural difference is the required DAV capability (`calendar-access` vs `addressbook`) and a CalDAV-only scheduling check.

Both checkers are **service-level**: they target the corresponding service published on a subdomain and are configured from that service's own **Checks** tab. Discovery follows RFC 6764: the checker resolves a context URL from `/.well-known/{caldav,carddav}` and from `_caldavs._tcp` / `_carddavs._tcp` SRV records (with an optional `path=` TXT hint), then probes that endpoint.

When no credentials are supplied, only the anonymous phase runs (discovery, transport, OPTIONS). Supplying a username and password unlocks the authenticated phase: principal discovery, home-set, collection enumeration and the REPORT probe.

{{% notice style="info" title="TLS posture is out of scope" %}}
These checkers confirm only that an HTTPS session can be established to the context URL. Certificate validation (chain, hostname, expiry, ciphers) is deliberately left to the dedicated TLS checker. See {{< relref "/reference/checkers/tls" >}}.
{{% /notice %}}

## What it checks

| Rule | What it verifies | Conditions |
|---|---|---|
| `dav_discovery` | A context URL resolves via `/.well-known` or SRV. | **Critical** if no context URL can be resolved. **Warning** when `/.well-known` returns `200` instead of a `301`/`302` redirect (legal but many clients won't follow it). **Info** when the URL was resolved via SRV but `/.well-known` is broken. |
| `dav_transport` | The HTTPS connection to the context URL is reachable. | **Critical** if the connection fails. |
| `dav_options` | An HTTP `OPTIONS` request advertises the required DAV capability (`calendar-access` for CalDAV, `addressbook` for CardDAV) and the `PROPFIND`/`REPORT` methods. | **Critical** if OPTIONS fails or the capability is missing. **Warning** if the `Allow` header lacks `PROPFIND` or `REPORT`. |
| `dav_principal` | The current-user principal URL is discovered via authenticated `PROPFIND`. | **Critical** on failure. **Unknown** when no credentials are supplied (phase skipped). |
| `dav_home_set` | The calendar/addressbook home-set is discovered from the principal. | **Critical** on failure. **Unknown** when skipped. |
| `dav_collections` | Calendar/addressbook collections enumerate and expose their properties (supported component set, supported address data, display name…). | **Critical** on failure. **Warning** if the home-set is empty. **Unknown** when skipped. |
| `dav_report` | The server accepts a minimal `calendar-query` / `addressbook-query` `REPORT` against the first collection. | **Critical** on failure. **Warning** if the query returns an unexpected response. **Unknown** when skipped. |
| `caldav_scheduling` | *(CalDAV only)* When `calendar-schedule` is advertised, the principal exposes `schedule-inbox-URL` and `schedule-outbox-URL`. | **Warning** if advertised but the URLs are missing or the probe fails. **Info** when scheduling is not advertised. **Unknown** when skipped. |

The HTML report surfaces the most common misconfigurations as callouts: `/.well-known` returning `200`, no SRV and no well-known (service unreachable), a plaintext SRV record without a secure counterpart, a server not advertising the required DAV class, and the authenticated phase being skipped for lack of credentials.

## Options

Both checkers accept the same options.

| Option | Meaning | Default |
|---|---|---|
| Username | Optional. Supplying credentials unlocks the authenticated checks (principal, home-set, collections, REPORT probe). | *(empty)* |
| Password or token | Optional. Paired with the username for HTTP Basic authentication. | *(empty)* |
| Explicit context URL | Optional. Bypasses `/.well-known` and SRV discovery; use for servers with a non-standard layout. | *(empty)* |
| Domain name | The domain to probe (auto-filled from the service scope). | *(auto)* |
| Timeout (seconds) | Per-request HTTP timeout. | `10` |

## In happyDomain

Enable the CalDAV or CardDAV checker from the **Checks** tab of a CalDAV or CardDAV service. The domain name is filled in automatically; add credentials only if you want the authenticated collection and REPORT checks to run. See {{< relref "/pages/checks" >}} for the full workflow, and {{< relref "/reference/checkers/tls" >}} for the certificate posture of the same endpoints.
