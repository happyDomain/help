---
data: 2023-01-19T19:31:08+02:00
title: Using Docker
weight: 15
---

happyDomain is sponsored by Docker.
You'll find the official container image on [the Docker Hub](https://hub.docker.com/r/happydomain/happydomain/).

The image runs happyDomain as a single process with a LevelDB database stored on disk — no extra database to configure.


## Supported tags and architectures

All tags are built for `amd64`, `arm64` and `arm/v7` and are based on Alpine.

Currently available tags:

- `latest`: the most up-to-date version, corresponding to the master branch.


## Quick start (single container)

For a quick test or personal use, pass `HAPPYDOMAIN_NO_AUTH=1` to skip account management:

```
docker run -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

Data are stored inside the container. To keep them across restarts, attach a volume:

```
docker volume create happydomain_data
docker run -e HAPPYDOMAIN_NO_AUTH=1 -v happydomain_data:/data -p 8081:8081 happydomain/happydomain
```

For a production single-container setup that sends e-mail:

```
docker run \
  -e HAPPYDOMAIN_MAIL_SMTP_HOST=smtp.yourcompany.com \
  -e HAPPYDOMAIN_MAIL_SMTP_USERNAME=happydomain \
  -e HAPPYDOMAIN_MAIL_SMTP_PASSWORD=secret \
  -v /var/lib/happydomain:/data \
  -p 8081:8081 \
  happydomain/happydomain
```


## Full deployment with all checkers

happyDomain ships every checker built-in, but several of them rely on
external tools (DNSViz, Zonemaster, Matrix federation tester) that are
packaged in their own container images. Running these as separate services
gives you the full checker experience and better isolation.

The recommended approach is `docker compose`. Save the following file as
`docker-compose.yml` and run `docker compose up -d`.

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

### How it works

Each checker runs as a standalone HTTP service. happyDomain delegates check
requests to the matching container via the `HAPPYDOMAIN_CHECKER_<ID>_ENDPOINT`
environment variable. When an endpoint is not set, the corresponding checker
runs locally inside the happyDomain process instead.

Two checkers rely on additional third-party backends:

- **Zonemaster** (`checker-zonemaster`) queries the `zonemaster/backend`
  service. The `HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL` variable tells
  the checker where that backend listens.
- **Matrix federation tester** (`checker-matrix`) queries the
  `matrixdotorg/federation-tester-backend` service. The
  `HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER` variable points to its
  report endpoint.

### Optional: happyDeliver

If you run a [happyDeliver](https://happydeliver.io) instance for mail-flow
monitoring, uncomment the `HAPPYDOMAIN_CHECKER_HAPPYDELIVER_ENDPOINT` line and
add the corresponding service:

```yaml
  checker-happydeliver:
    image: happydomain/checker-happydeliver
    restart: unless-stopped
```

### Optional: blacklist API keys

The `checker-blacklist` service works without API keys (it uses DNS-based
blocklists by default), but you can enable additional sources — Google Safe
Browsing, VirusTotal, abuse.ch URLhaus — by configuring the matching admin
options from the happyDomain administration interface once the stack is running.


## Admin interface

happyDomain exposes administration commands through a Unix socket. The
container includes the `hadmin` wrapper:

```
docker exec my_container hadmin /api/users
docker exec my_container hadmin /api/users/0123456789/send_validation_email -X POST
```

`hadmin` is a thin wrapper around `curl` — start with the URL path, then add
any `curl` options after it.


## Using a configuration file

Instead of environment variables, you can place a configuration file either in
`/data/happydomain.conf` (inside the data volume) or bind-mount it to
`/etc/happydomain.conf`:

```
docker run -v happydomain.conf:/etc/happydomain.conf -p 8081:8081 happydomain/happydomain
```
