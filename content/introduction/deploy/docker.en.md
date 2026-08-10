---
data: 2023-01-19T19:31:08+02:00
title: Using Docker
weight: 15
---

happyDomain is sponsored by Docker.
You'll find the official container image on [the Docker Hub](https://hub.docker.com/r/happydomain/happydomain/).

The image runs happyDomain as a single process with a LevelDB database stored on disk: no extra database to configure. **Every checker shipped with happyDomain is built into the binary**, so a single container is already a complete, fully featured happyDomain.


## Supported tags and architectures

All tags are built for `amd64`, `arm64` and `arm/v7` and are based on Alpine.

Currently available tags:

- `latest`: the most up-to-date version, corresponding to the master branch.

Tags suffixed with **`-cgo`** contain a dynamically linked build, the only one
able to load [plugins]({{% relref "/reference/plugins" %}}). Use them only if
you need that: the default tags are statically built, smaller and more
portable, but they ignore any plugin directory.


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


## Recommended deployment: `docker compose`

This is the setup we recommend for **almost every deployment**, from a personal
instance to a company-wide one. It is the same file you'll find at the root of
[the happyDomain repository](https://github.com/happyDomain/happydomain/blob/master/docker-compose.yml).

Beside happyDomain itself, it runs the few third-party backends that some
checkers need to reach (DNSViz, Zonemaster and the Matrix federation tester),
plus a local recursive resolver so that DNS lookups don't depend on your
hosting provider's resolver.

Save the file as `docker-compose.yml` and run `docker compose up -d`.

```yaml
name: happydomain

services:
  happydomain:
    image: happydomain/happydomain
    ports:
      - "8081:8081"
    environment:
      # Uncomment for single-user / testing
      # HAPPYDOMAIN_NO_AUTH: "1"

      # Mail configuration (required for multi-user production use)
      # HAPPYDOMAIN_MAIL_SMTP_HOST: "mailer"

      # Use the local helper backends instead of the public ones
      HAPPYDOMAIN_CHECKER_DNSVIZ_ENDPOINT: "http://dnsviz:8080"
      HAPPYDOMAIN_CHECKER_ZONEMASTER_ZONEMASTERAPIURL: "http://zonemaster:5000"
      HAPPYDOMAIN_CHECKER_MATRIXIM_FEDERATIONTESTERSERVER: "http://matrixfederationtester:8080/api/report?server_name=%s"

    dns:
      - ${HAPPYDOMAIN_DNS_IP:-172.28.0.53}
    restart: unless-stopped
    volumes:
      - storage:/data:rw

  # Local recursive resolver, so DNS checks don't rely on a third-party resolver
  unbound:
    image: alpinelinux/unbound
    restart: unless-stopped
    configs:
      - source: unbound_conf
        target: /etc/unbound/unbound.conf
        uid: "100"
        gid: "101"
    networks:
      default:
        ipv4_address: ${HAPPYDOMAIN_DNS_IP:-172.28.0.53}

  # DNSViz analysis backend, used by the DNSSEC visualisation checker
  dnsviz:
    image: happydomain/checker-dnsviz
    restart: unless-stopped

  # Zonemaster backend, used by the Zonemaster checker
  zonemaster:
    image: zonemaster/backend
    command: full
    restart: unless-stopped

  # Matrix federation tester, used by the Matrix checker
  matrixfederationtester:
    image: matrixdotorg/federation-tester-backend
    environment:
      BIND_ADDRESS: "0.0.0.0:8080"
    restart: unless-stopped

configs:
  unbound_conf:
    content: |
      server:
          verbosity: 1
          interface: 0.0.0.0
          port: 53
          do-ip4: yes
          do-ip6: no
          do-udp: yes
          do-tcp: yes

          access-control: 127.0.0.0/8 allow
          access-control: ${HAPPYDOMAIN_SUBNET:-172.28.0.0/24} allow

          cache-max-ttl: 60

          so-sndbuf: 0
          so-rcvbuf: 0

          trust-anchor-file: "/etc/unbound/root.key"

volumes:
  storage:

networks:
  default:
    ipam:
      config:
        - subnet: ${HAPPYDOMAIN_SUBNET:-172.28.0.0/24}
```

With this stack, **all the checkers shipped with happyDomain are available**:
they run inside the happyDomain process, and the three services above only
provide the external analysis engines two or three of them rely on.


## Where do checkers run?

There are two separate questions here, and mixing them up is a common source of
confusion.

**First, how does happyDomain know a checker exists?** Only two answers:

- **built-in**: the checker is compiled into the happyDomain binary. Every
  checker we ship is available this way, and it's what you get with the
  deployments above;
- **plugin**: a shared library (`.so`) dropped into a directory passed with
  `-plugins-directory`, loaded at start-up. This is the only way to add a
  checker that is *not* part of happyDomain (one you wrote for your own needs,
  or one from a third party), without patching and rebuilding the server. It
  requires a **`-cgo` image tag**: the default image is a static build with no
  plugin support (see [Plugins]({{% relref "/reference/plugins" %}})).

**Then, where does the work actually run?** A registered checker runs inside the
happyDomain process by default. If you set
`HAPPYDOMAIN_CHECKER_<ID>_ENDPOINT`, it instead delegates the collection to a
**standalone container** exposing that checker over HTTP.

The important consequence: a checker container is *not* a way to add a checker.
The `endpoint` setting only exists for checkers happyDomain already knows about,
so a home-made checker must first be loaded as a plugin, and only then may you
point it at a container.

For a given checker, running it locally or delegating it to a container performs
exactly the same checks and produces the same results. Delegating buys you
process isolation and the ability to scale a single checker independently; it
costs you a lot more RAM, more moving parts and a much less pleasant debugging
experience when one of them misbehaves.

Unless you're operating at a scale where that trade-off pays off, or you need a
checker that isn't shipped with happyDomain, stay with the compose file above.
If you do need it, see
[Running checkers as separate containers]({{% relref "checkers-containers" %}}).


## Updating the stack

**1. Check whether the reference `docker-compose.yml` has changed.** Upgrading
the images is not always enough: a new version may come with a new service (a
checker that needs its own backend, for instance) or new settings to declare.
Compare your file with the one in
[the happyDomain repository](https://github.com/happyDomain/happydomain/blob/master/docker-compose.yml),
and port the changes that concern you into your own copy:

```
curl -sO https://raw.githubusercontent.com/happyDomain/happydomain/master/docker-compose.yml
diff -u docker-compose.yml /path/to/your/docker-compose.yml
```

**2. Pull the images and recreate the containers.**

```
docker compose up -d --pull always
```

`--pull always` fetches the latest image for every service before recreating
the ones that actually changed; the others are left untouched. Your data live
in the `storage` volume and survive the operation.

Then check that everything came back up, and reclaim the disk space taken by
the replaced images:

```
docker compose ps
docker compose logs -f happydomain
docker image prune
```

For a single container started with `docker run`, the equivalent is to pull the
image, remove the old container and start it again with the same options (your
volume keeps the data):

```
docker pull happydomain/happydomain
docker rm -f my_container
```


## Admin interface

happyDomain exposes administration commands through a Unix socket. The
container includes the `hadmin` wrapper:

```
docker exec my_container hadmin /api/users
docker exec my_container hadmin /api/users/0123456789/send_validation_email -X POST
```

`hadmin` is a thin wrapper around `curl`: start with the URL path, then add
any `curl` options after it.


## Using a configuration file

Instead of environment variables, you can place a configuration file either in
`/data/happydomain.conf` (inside the data volume) or bind-mount it to
`/etc/happydomain.conf`:

```
docker run -v happydomain.conf:/etc/happydomain.conf -p 8081:8081 happydomain/happydomain
```
