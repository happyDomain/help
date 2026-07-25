---
data: 2024-06-26T20:44:25+02:00
title: Connect to a BIND server
weight: 30
---

[BIND](https://www.isc.org/bind/) is an authoritative and recursive DNS server developed by the [Internet Systems Consortium](https://isc.org).

It is possible to use it with happyDomain through [Dynamic DNS (RFC 2136)](https://www.rfc-editor.org/rfc/rfc2136), which works with a remote server, or by editing the zone files directly, when happyDomain runs on the same host.

This documentation will guide you through configuring BIND to enable Dynamic DNS and connect your domains to happyDomain, then through the zone file alternative.


## Configure BIND to enable Dynamic DNS

First, you need to edit the main BIND configuration file (usually `/etc/named.conf` or `/etc/bind/named.conf` depending on your distribution) to add a secret that will be shared between happyDomain and BIND to authenticate the changes. Then you must indicate which domains will be managed by happyDomain.

### Adding a Shared Secret

Under the main `key` section of your configuration, add the following key:

```conf
key "happydomain" {
    algorithm hmac-sha512;
    secret "<SOME_SECRET>";
};
```

Replace `<SOME_SECRET>` with a string obtained using `openssl rand -base64 48`.

### Creating an Authorization Rule for happyDomain

In addition to the key, you must specify how the key can be used by defining an ACL and allowing updates from it.

Add the following ACL to your configuration:

```conf
acl "happydomain_acl" {
    key happydomain;
};
```

### Allowing Updates for Each Zone

Now that you have created a rule allowing the `happydomain` key to make changes, you need to indicate to which zones this rule applies.

For each zone, you must add an `update-policy` statement referencing the `happydomain_acl` ACL:

For example, for an existing `happydomain.org` zone, add the `update-policy` statement as follows:

```conf
zone "happydomain.org" {
    type master;
    file "/var/named/happydomain.org.db";
    update-policy {
        grant happydomain_acl name happydomain.org. ANY;
    };
};
```

The `update-policy` statement is a list, so you may already have other policies in this list. In this case, just add the `grant` statement for `happydomain_acl`.

### Allowing Updates for All Zones

If you manage many zones, it may be more convenient to set the default authorization for all zones. In this case, you can use a `global` `update-policy` in the `options` section:

```conf
options {
    update-policy {
        grant happydomain_acl zonesub ANY;
    };
};
```

This will apply the `update-policy` to all zones, allowing the `happydomain_acl` to update any record.

### Apply the Configuration

After modifying the configuration file, reload the BIND service to apply the changes:

```sh
rndc reload
```

## Link happyDomain and BIND

Once BIND is well configured, you can link it to happyDomain using [the *Dynamic DNS* connector]({{% ref "/pages/provider-new-choice.md" %}}) :

![The Dynamic DNS connector on the host selection page](/img/choose-dynamic-dns.png)

Follow these steps:

1. Navigate to the Dynamic DNS connector on the host selection page in happyDomain.
2. Fill in the form with the address where your BIND server is accessible.
3. Fill in the Key fields with the information from the `key` section in the BIND configuration:
    - **Key Name**: corresponds to the key name in BIND's configuration (e.g., `happydomain`).
    - **Key Algorithm**: corresponds to the algorithm (e.g., `hmac-sha512`).
    - **Secret Key**: corresponds to the secret.

Once the provider is added, it does not allow you to list existing domains, but you can still manually add all your domains.

By following these steps, you will have configured BIND to work with happyDomain using Dynamic DNS, ensuring secure and authenticated DNS updates.


## Editing zone files directly

When happyDomain runs on the same host as BIND, it can also edit the zone files on disk instead of going through Dynamic DNS. This is the *Bind files/RFC 1035* connector.

As it gives access to the local file system, this connector is disabled by default and has to be enabled by the administrator of the instance, who must declare which directory it may use:

```
./happydomain -with-bind-provider /etc/named/zones
```

Users are confined to that directory and its subdirectories, so they cannot read or overwrite a file located elsewhere on the host. See [the `with-bind-provider` option]({{% relref "config#bind" %}}) for the details, including how to allow several directories.

Once the option is set, the connector is available on the host selection page and asks for two fields:

- **Directory**: where the zone files are stored. It has to be the allowed directory, or one of its subdirectories.
- **File format**: how the file of a zone is named, `%U.zone` by default. It must remain a bare file name, without any path separator. The available placeholders are documented in [the dnscontrol documentation](https://docs.dnscontrol.org/service-providers/providers/bind#filenameformat).

Remember that BIND does not reload a zone file on its own: after publishing your changes, run `rndc reload <zone>` (or configure BIND to watch the files) for them to be served.
