---
data: 2023-01-19T19:31:08+02:00
title: Using Docker
weight: 15
---

happyDomain is sponsored by Docker.
You'll find the official container image on [the Docker Hub](https://hub.docker.com/r/happydomain/happydomain/).

This image will run happyDomain as a single process, with a LevelDB database (similarly to sqlite, LevelDB is stored on disk, no need to configure anything).


## Supported tags and architectures

All tags are build for `amd64`, `arm64` and `arm/v7` and are based on alpine.

Currently, available tags are:

- `latest`: this is a the most up to date version, corresponding to the master branch.

## Using this image

### For testing purpose

You can test happyDomain or use it for your own usage, with the option `HAPPYDOMAIN_NO_AUTH=1`: this will automatically creates a default account, and disable all features related to the user management (signup, login. ...).

```
docker run -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

Data are stored in `/data` directory. If you want to keep your settings from one run to another, you'll need to attach this directory to a Docker managed volume or to a directory on your host:

```
docker volume create happydomain_data
docker run -e HAPPYDOMAIN_NO_AUTH=1 -v happydomain_data:/data -p 8081:8081 happydomain/happydomain
```

### In production

happyDomain needs to send e-mail, in order to verify addresses and doing password recovery, so you need basically to configure a SMTP relay.
Use the options `HAPPYDOMAIN_MAIL_SMTP_HOST`, `HAPPYDOMAIN_MAIL_SMTP_PORT` (default 25), `HAPPYDOMAIN_MAIL_SMTP_USERNAME` and `HAPPYDOMAIN_MAIL_SMTP_PASSWORD` for this purpose:

```
docker run -e HAPPYDOMAIN_MAIL_SMTP_HOST=smtp.yourcompany.com -e HAPPYDOMAIN_MAIL_SMTP_USERNAME=happydomain -e HAPPYDOMAIN_MAIL_SMTP_PASSWORD=secret -v /var/lib/happydomain:/data -p 8081:8081 happydomain/happydomain
```

By default, happyDomain uses `sendmail`, if you prefer, you can create you own image with the package `ssmtp`:

```
FROM happydomain/happydomain
RUN apk --no-cache add ssmtp
COPY my_ssmtp.conf /etc/ssmtp/ssmtp.conf
```

If you prefer using a configuration file, you can place it either in `/data/happydomain.conf` to use the volume, or bind your file to `/etc/happydomain.conf`:

```
docker run -v happydomain.conf:/etc/happydomain.conf -p 8081:8081 happydomain/happydomain
```

## Admin Interface

happyDomain exposes some administration command through a unix socket. The docker container contains a script to access this admin part: `hadmin`.

You can use it this way:

```
docker exec my_container hadmin /api/users
docker exec my_container hadmin /api/users/0123456789/send_validation_email -X POST
```

This is in fact a wrapper above `curl`, but you have to start by the URL, and place options after it.
