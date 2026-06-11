---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: HTTP / HTTPS
description: "Probes a web server over HTTP and HTTPS and evaluates reachability, the redirect chain, and the full battery of HTTP security response headers and cookie hygiene rules."
weight: 180
---

The **HTTP / HTTPS** checker probes the web server declared by a *Server* service over plain HTTP (port 80) and HTTPS (port 443), then evaluates a battery of independent rules on the responses: reachability, the HTTP→HTTPS redirect chain, and the modern set of HTTP security headers (HSTS, CSP, frame options, cross-origin isolation…) along with cookie hygiene.

**Scope:** service-level. It attaches to services of type `abstract.Server` (a subdomain that publishes `A`/`AAAA` records) and is configured from that service's **Checks** tab.

Deep TLS and certificate analysis is intentionally delegated to the {{< relref "/reference/checkers/tls" >}} checker; this checker relies on TLS only as a transport.

## What it checks

| Rule | Verifies | Severity |
|------|----------|----------|
| `http.tcp_reachable` | Every probed IP accepts an HTTP connection on port 80. | Critical |
| `https.tcp_reachable` | Every probed IP accepts an HTTPS connection on port 443. | Critical |
| `http.https_redirect` | Plain HTTP redirects to an HTTPS URL on the same host. | Warning |
| `http.redirect_chain` | The redirect chain has no loops, excessive length, or scheme downgrades. | Warning |
| `http.redirect_permanence` | HTTP→HTTPS upgrade uses 301 or 308 (permanent) rather than 302/307. | Warning |
| `http.hsts` | Presence and quality of the Strict-Transport-Security header on HTTPS. | Warning |
| `http.csp` | Presence and quality of the Content-Security-Policy header on HTTPS. | Warning |
| `http.x_frame_options` | Responses set X-Frame-Options or a CSP `frame-ancestors` directive. | Warning |
| `http.x_content_type_options` | Responses set `X-Content-Type-Options: nosniff`. | Warning |
| `http.x_xss_protection` | Reports the legacy X-XSS-Protection header value (CSP is the proper replacement). | Info |
| `http.referrer_policy` | Responses set a privacy-preserving Referrer-Policy header. | Warning |
| `http.permissions_policy` | The Permissions-Policy header restricts powerful APIs (camera, microphone, geolocation…). | Warning |
| `http.coop` | The Cross-Origin-Opener-Policy header is set for cross-origin process isolation. | Warning |
| `http.coep` | The Cross-Origin-Embedder-Policy header is set (required with COOP for cross-origin isolation). | Warning |
| `http.corp` | The Cross-Origin-Resource-Policy header restricts cross-origin embedding. | Warning |
| `http.cookie_flags` | Cookies set over HTTPS use the Secure, HttpOnly and SameSite attributes. | Warning |
| `http.cookie_prefixes` | Cookies using the `__Secure-` / `__Host-` prefixes meet the RFC 6265bis constraints. | Warning |
| `http.cookie_size` | Flags Set-Cookie lines exceeding the 4096-byte minimum browsers must support. | Warning |
| `http.sri` | Reports cross-origin script/style tags missing Subresource Integrity attributes. | Warning |
| `http.security_txt` | Reports whether `/.well-known/security.txt` (RFC 9116) is published. | Warning |

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Per-request timeout (ms) (`probeTimeoutMs`) | Maximum time allowed for a single HTTP/HTTPS request. | 10000 |
| Max redirects to follow (`maxRedirects`) | Stop following redirects after this many hops. | 5 |
| User-Agent (`userAgent`) | User-Agent header sent with every request. | `happyDomain-checker-http/1.0` |
| Require HTTPS (`requireHTTPS`) | Plain HTTP must redirect to HTTPS. | true |
| Require HSTS (`requireHSTS`) | HTTPS responses must include a Strict-Transport-Security header. | true |
| Min HSTS max-age (days) (`minHSTSMaxAgeDays`) | Minimum acceptable HSTS max-age, in days. | 180 |
| Require Content-Security-Policy (`requireCSP`) | HTTPS responses must include a Content-Security-Policy header. | false |

## In happyDomain

This is a service-level checker: configure it from the **Checks** tab of the *Server* service on the relevant subdomain. For deep certificate posture, add the {{< relref "/reference/checkers/tls" >}} checker as well. For the general workflow of configuring and reading checks, see {{< relref "/pages/checks" >}}.
