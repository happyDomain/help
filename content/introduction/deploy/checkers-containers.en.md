---
data: 2026-08-01T10:00:00+02:00
title: Running checkers as separate containers
weight: 17
---

{{% notice style="warning" title="This is not the default deployment" %}}
You almost certainly don't need this page. The compose file described in
[Using Docker]({{% relref "docker" %}}) already gives you **every checker we
ship**, with the exact same results. Splitting them into containers adds no
feature; it only changes where they run. (Adding a checker that is *not* part
of happyDomain is a different question, and the answer there is usually a
[plugin]({{% relref "/reference/plugins" %}}).)
{{% /notice %}}


## Why this exists

happyDomain has to **know** a checker before it can run it. There are exactly
two ways to register one:

- **built-in**, compiled into the happyDomain binary. Every checker we ship is
  available this way, and it is the default;
- **plugin**, a shared library loaded at start-up from a directory given with
  `-plugins-directory`. This is the only way to register a checker that isn't
  part of happyDomain, such as one written in-house. It requires a `-cgo` build
  of happyDomain and a platform where Go supports plugins.

Once a checker is registered, you can decide **where its work runs**: inside the
happyDomain process (the default), or in a **standalone container** it delegates
to over HTTP. That is what this page is about.

{{% notice style="note" title="A container does not register a checker" %}}
Running `happydomain/checker-<something>` and setting its endpoint only works
because that checker is already known to happyDomain. Pointing an endpoint at a
checker the server has never heard of does nothing: a custom checker must be
loaded as a plugin first, and only then may it be delegated to a container.
{{% /notice %}}

Take [`checker-ping`](https://framagit.org/happyDomain/checker-ping), which
verifies that every IP address of a zone answers ICMP within a given delay. It
is already inside your happyDomain binary, whichever deployment you choose. If
you also run the `happydomain/checker-ping` container and set
`HAPPYDOMAIN_CHECKER_PING_ENDPOINT`, the built-in checker simply forwards the
work to that container instead of doing it itself. Same checks, same rules,
same output.


## When it's worth it

The split deployment makes sense when:

- you monitor **thousands of zones** and want to scale a hot checker
  horizontally, independently from the rest of happyDomain;
- you need **strong process isolation** between checks, for example because a
  checker needs elevated network capabilities (`NET_RAW` for ICMP) that you
  don't want granted to the main process;
- you want to **pin, upgrade or roll back a single checker** on its own release
  cycle.


## What it costs

- **Memory.** Around thirty extra containers, each with its own runtime, for a
  feature set strictly equal to the single-container one.
- **Operations.** Thirty more images to pull, watch and upgrade.
- **Debugging.** When a check fails, the cause may now be the checker, the
  container, the network between them, or a stale endpoint variable. That
  investigation is significantly more painful than reading a single log stream.

If none of the reasons above applies to you, go back to
[Using Docker]({{% relref "docker" %}}).


## How delegation works

`HAPPYDOMAIN_CHECKER_<ID>_ENDPOINT` is an option **of a registered checker**,
not a way to declare one. For every checker happyDomain knows about (built-in
or loaded from a plugin), setting that variable makes it forward the collection
to the given URL instead of doing the work itself; leaving it empty keeps the
checker running locally inside the happyDomain process. `<ID>` is the checker's
own identifier; if it doesn't match a registered checker, the variable is
simply ignored.

You can mix both freely: delegate only the checkers you actually want to
isolate, and leave the rest running in-process.

Two checkers rely on additional third-party backends, independently of this
choice:

- **Zonemaster** (`checker-zonemaster`) queries the `zonemaster/backend`
  service. The `HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL` variable tells
  the checker where that backend listens.
- **Matrix federation tester** (`checker-matrix`) queries the
  `matrixdotorg/federation-tester-backend` service. The
  `HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER` variable points to its
  report endpoint.


## Full compose file

```yaml
services:
  happydomain:
    image: happydomain/happydomain
    ports:
      - "8080:8081"
    environment:
      # Uncomment for single-user / testing
      # HAPPYDOMAIN_NO_AUTH: "1"

      # Mail configuration (required for multi-user production use)
      # HAPPYDOMAIN_MAIL_SMTP_HOST: "mailer"

      # ── DNS / DNSSEC ─────────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_DNSVIZ_ENDPOINT: "http://checker-dnsviz:8080"
      HAPPYDOMAIN_CHECKER_DNSSEC_ENDPOINT: "http://checker-dnssec:8080"
      HAPPYDOMAIN_CHECKER_ZONEMASTER_ENDPOINT: "http://checker-zonemaster:8080"
      HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL: "http://zonemaster:5000"
      HAPPYDOMAIN_CHECKER_DELEGATION_ENDPOINT: "http://checker-delegation:8080"
      HAPPYDOMAIN_CHECKER_AUTHORITATIVE_CONSISTENCY_ENDPOINT: "http://checker-authoritative-consistency:8080"
      HAPPYDOMAIN_CHECKER_ALIAS_ENDPOINT: "http://checker-alias:8080"
      HAPPYDOMAIN_CHECKER_LEGACY_RECORDS_ENDPOINT: "http://checker-legacy-records:8080"
      HAPPYDOMAIN_CHECKER_NS_RESTRICTIONS_ENDPOINT: "http://checker-ns-restrictions:8080"
      HAPPYDOMAIN_CHECKER_RESOLVER_PROPAGATION_ENDPOINT: "http://checker-resolver-propagation:8080"
      HAPPYDOMAIN_CHECKER_REVERSE_ZONE_ENDPOINT: "http://checker-reverse-zone:8080"
      HAPPYDOMAIN_CHECKER_PTR_ENDPOINT: "http://checker-ptr:8080"
      HAPPYDOMAIN_CHECKER_DANGLING_ENDPOINT: "http://checker-dangling:8080"

      # ── Security / Certificates ───────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_TLS_ENDPOINT: "http://checker-tls:8080"
      HAPPYDOMAIN_CHECKER_DANE_ENDPOINT: "http://checker-dane:8080"
      HAPPYDOMAIN_CHECKER_CAA_ENDPOINT: "http://checker-caa:8080"
      HAPPYDOMAIN_CHECKER_BLACKLIST_ENDPOINT: "http://checker-blacklist:8080"

      # ── E-mail ────────────────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_SMTP_ENDPOINT: "http://checker-smtp:8080"
      HAPPYDOMAIN_CHECKER_EMAIL_AUTOCONFIG_ENDPOINT: "http://checker-email-autoconfig:8080"
      HAPPYDOMAIN_CHECKER_OPENPGPKEY_SMIMEA_ENDPOINT: "http://checker-email-keys:8080"

      # ── Web & Protocols ───────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_HTTP_ENDPOINT: "http://checker-http:8080"
      HAPPYDOMAIN_CHECKER_SSH_ENDPOINT: "http://checker-ssh:8080"
      HAPPYDOMAIN_CHECKER_PING_ENDPOINT: "http://checker-ping:8080"
      HAPPYDOMAIN_CHECKER_SRV_ENDPOINT: "http://checker-srv:8080"

      # ── Collaboration / Messaging ─────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_MATRIXIM_ENDPOINT: "http://checker-matrix:8080"
      HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER: "http://matrixfederationtester:8080/api/report?server_name=%s"
      HAPPYDOMAIN_CHECKER_XMPP_ENDPOINT: "http://checker-xmpp:8080"
      HAPPYDOMAIN_CHECKER_SIP_ENDPOINT: "http://checker-sip:8080"

      # ── Directory & Auth ──────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_LDAP_ENDPOINT: "http://checker-ldap:8080"
      HAPPYDOMAIN_CHECKER_KERBEROS_ENDPOINT: "http://checker-kerberos:8080"
      HAPPYDOMAIN_CHECKER_STUNTURN_ENDPOINT: "http://checker-stun-turn:8080"

      # ── CalDAV / CardDAV ──────────────────────────────────────────────────
      HAPPYDOMAIN_CHECKER_CALDAV_ENDPOINT: "http://checker-caldav:8080"
      HAPPYDOMAIN_CHECKER_CARDDAV_ENDPOINT: "http://checker-carddav:8080"

      # ── Optional: happyDeliver integration ────────────────────────────────
      # HAPPYDOMAIN_CHECKER_HAPPYDELIVER_ENDPOINT: "http://checker-happydeliver:8080"

    restart: unless-stopped
    volumes:
      - storage:/var/lib/happydomain:rw

  # ── DNS / DNSSEC checkers ──────────────────────────────────────────────────

  checker-dnsviz:
    image: happydomain/checker-dnsviz
    restart: unless-stopped

  checker-dnssec:
    image: happydomain/checker-dnssec
    restart: unless-stopped

  checker-zonemaster:
    image: happydomain/checker-zonemaster
    restart: unless-stopped

  zonemaster:
    image: zonemaster/backend
    command: full
    restart: unless-stopped

  checker-delegation:
    image: happydomain/checker-delegation
    restart: unless-stopped

  checker-authoritative-consistency:
    image: happydomain/checker-authoritative-consistency
    restart: unless-stopped

  checker-alias:
    image: happydomain/checker-alias
    restart: unless-stopped

  checker-legacy-records:
    image: happydomain/checker-legacy-records
    restart: unless-stopped

  checker-ns-restrictions:
    image: happydomain/checker-ns-restrictions
    restart: unless-stopped

  checker-resolver-propagation:
    image: happydomain/checker-resolver-propagation
    restart: unless-stopped

  checker-reverse-zone:
    image: happydomain/checker-reverse-zone
    restart: unless-stopped

  checker-ptr:
    image: happydomain/checker-ptr
    restart: unless-stopped

  checker-dangling:
    image: happydomain/checker-dangling
    restart: unless-stopped

  # ── Security / Certificate checkers ───────────────────────────────────────

  checker-tls:
    image: happydomain/checker-tls
    restart: unless-stopped

  checker-dane:
    image: happydomain/checker-dane
    restart: unless-stopped

  checker-caa:
    image: happydomain/checker-caa
    restart: unless-stopped

  checker-blacklist:
    image: happydomain/checker-blacklist
    restart: unless-stopped

  # ── E-mail checkers ────────────────────────────────────────────────────────

  checker-smtp:
    image: happydomain/checker-smtp
    restart: unless-stopped

  checker-email-autoconfig:
    image: happydomain/checker-email-autoconfig
    restart: unless-stopped

  checker-email-keys:
    image: happydomain/checker-email-keys
    restart: unless-stopped

  # ── Web & Protocol checkers ────────────────────────────────────────────────

  checker-http:
    image: happydomain/checker-http
    restart: unless-stopped

  checker-ssh:
    image: happydomain/checker-ssh
    restart: unless-stopped

  checker-ping:
    image: happydomain/checker-ping
    restart: unless-stopped
    cap_add:
      - NET_RAW  # required for ICMP ping

  checker-srv:
    image: happydomain/checker-srv
    restart: unless-stopped

  # ── Collaboration / Messaging checkers ─────────────────────────────────────

  checker-matrix:
    image: happydomain/checker-matrix
    restart: unless-stopped

  matrixfederationtester:
    image: matrixdotorg/federation-tester-backend
    environment:
      BIND_ADDRESS: "0.0.0.0:8080"
    restart: unless-stopped

  checker-xmpp:
    image: happydomain/checker-xmpp
    restart: unless-stopped

  checker-sip:
    image: happydomain/checker-sip
    restart: unless-stopped

  # ── Directory & Auth checkers ──────────────────────────────────────────────

  checker-ldap:
    image: happydomain/checker-ldap
    restart: unless-stopped

  checker-kerberos:
    image: happydomain/checker-kerberos
    restart: unless-stopped

  checker-stun-turn:
    image: happydomain/checker-stun-turn
    restart: unless-stopped

  # ── CalDAV / CardDAV checkers ──────────────────────────────────────────────

  checker-caldav:
    image: happydomain/checker-caldav
    restart: unless-stopped

  checker-carddav:
    image: happydomain/checker-carddav
    restart: unless-stopped

volumes:
  storage:
```


## Updating the stack

The procedure is the same as for the standard deployment
([Updating the stack]({{% relref "docker#updating-the-stack" %}})): first check
whether the file you started from has changed, then

```
docker compose up -d --pull always
```

Be aware that here, step one is on you: this file is not generated. When we ship
a **new checker**, its built-in version arrives with the happyDomain image, but
nothing adds the matching container and its `_ENDPOINT` variable to your file.
Re-read this page after each upgrade and port the new services you want to keep
delegating; a checker you don't list simply keeps running in-process, which is a
perfectly valid outcome.


## Optional: happyDeliver

If you run a [happyDeliver](https://happydeliver.io) instance for mail-flow
monitoring, uncomment the `HAPPYDOMAIN_CHECKER_HAPPYDELIVER_ENDPOINT` line and
add the corresponding service:

```yaml
  checker-happydeliver:
    image: happydomain/checker-happydeliver
    restart: unless-stopped
```


## Optional: blacklist API keys

The `checker-blacklist` service works without API keys (it uses DNS-based
blocklists by default), but you can enable additional sources (Google Safe
Browsing, VirusTotal, abuse.ch URLhaus) by configuring the matching admin
options from the happyDomain administration interface once the stack is running.
