---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Matrix federation
description: "Verifies that a Matrix homeserver federates correctly, using the Matrix Federation Tester."
weight: 300
---

The **Matrix federation** checker verifies that a Matrix homeserver is correctly set up to federate with the rest of the Matrix network. It delegates the actual probing to a [Matrix Federation Tester](https://federationtester.matrix.org/) instance, stores the full report as an observation, and renders a rich HTML summary covering connections, certificates, well-known delegation and DNS/SRV resolution.

This is a **service-level** checker. It applies to services of type **Matrix** (instant messaging) and is configured from that service's own **Checks** tab.

## What it checks

| Rule | What it verifies | Severity |
|------|------------------|----------|
| `matrix.connection_reachable` | Every discovered federation endpoint accepts an inbound connection. | Critical |
| `matrix.federation_ok` | The overall federation status reported by the Matrix Federation Tester. | Critical |
| `matrix.srv_records` | The Matrix SRV lookup (`_matrix-fed._tcp` / `_matrix._tcp`) succeeded or was legitimately skipped. | Critical |
| `matrix.tls_checks` | The TLS posture on every reachable federation endpoint (certificate chain, hostname, Ed25519 key). | Critical |
| `matrix.version` | The homeserver answers `/_matrix/federation/v1/version` with its name and version. | Warning |
| `matrix.well_known` | `/.well-known/matrix/server`, when published, is valid and points at the declared `server_name`. | Critical |

## Options

| Option | Meaning | Default |
|--------|---------|---------|
| Matrix domain | The Matrix `server_name` to test. Filled in automatically from the service. | `matrix.org` |

An additional **admin-level** option, `federationTesterServer`, sets the URL template of the Federation Tester instance to query. It is configured by the happyDomain operator, not per check, and defaults to `https://federationtester.matrix.org/api/report?server_name=%s`.

## In happyDomain

Enable this checker from the **Checks** tab of a Matrix service; see {{< relref "/pages/checks" >}} for how to configure and schedule checks. The Matrix domain is filled in automatically from the service.
