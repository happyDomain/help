---
data: 2023-01-19T18:28:25+02:00
title: Testing or local single-user environment
weight: 10
---

Follow this installation guide if you want to use happyDomain directly on your machine, as simply as possible, without user management.

This is a simple configuration, suitable for evaluating happyDomain or for using it without authentication on your own machine, or with authentication provided by a reverse proxy.

You can deploy it using Docker or by downloading one of the available executables. Both methods are described below.


## Via Docker/podman

If you are familiar with Docker (or `podman`), you can have happyDomain up and running in seconds with the following command:

```
docker container run -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

If you want to make the data persistent, you need to add a volume:

```
docker volume create happydns_data
docker container run -v happydns_data:/data -e HAPPYDOMAIN_NO_AUTH=1 -p 8081:8081 happydomain/happydomain
```

Whatever your choice, go to <http://localhost:8081> to start using happyDomain.


## Via the executable

1. Start by downloading the executable matching your processor architecture from: <https://get.happydomain.org/master/>.
  The `darwin` versions are for macOS, while the `linux` versions are for GNU/Linux. All distributed versions are static and should work regardless of your libc (GNU libc in most cases, musl for Alpine, ...).

1. Make the binary you downloaded executable: `chmod +x happydomain-OS-ARCH`.

1. Run the binary: `HAPPYDOMAIN_NO_AUTH=1 ./happydomain-OS-ARCH`.

1. Go to <http://localhost:8081/> to start using happyDomain.


## What do these options do?

The `HAPPYDOMAIN_NO_AUTH=1` option is a parameter that tells happyDomain not to require authentication: domains are shared among all incoming connections. In practice, this automatically creates a default user and disables all login, account registration, and related features.


## Can I expose it on the Internet like this?

No! It is essential that you do not expose your happyDomain instance on the Internet without authentication.
Without it, all its content would be accessible to anyone, and they could take control of your domain(s).

Always use a reverse proxy such as nginx, Apache, HAproxy, Traefik, ... adding a filtering or basic authentication step.

For example, you can filter the IPs that can access the service.
Here is a sample nginx configuration filtering IPs (using the [`allow`](http://nginx.org/en/docs/http/ngx_http_access_module.html) directive):

```
server {
    listen 80;
    listen [::]:80;

    server_name happydomain.example.com;

    location / {
        allow 42.42.42.42;
        deny all;

        proxy_pass	http://localhost:8081;
        proxy_set_header  X-Forwarded-For  $proxy_add_x_forwarded_for;
    }

}

```

You can also add [basic password authentication](http://nginx.org/en/docs/http/ngx_http_auth_basic_module.html).
