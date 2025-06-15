---
data: 2023-02-09T09:12:25+01:00
title: Connect to a local knot
weight: 31
---

[Knot](https://knot-dns.cz) is an authoritative DNS server developed by the [cz.nic](https://nic.cz) association.

It is possible to use it with happyDomain through [Dynamic DNS (RFC 2136)](https://www.rfc-editor.org/rfc/rfc2136).


## Configure Knot to enable Dynamic DNS

First, you have to edit the main knot configuration file (usually `/etc/knot/knot.conf`) to add a secret that will be shared between happyDomain and knot to authenticate the changes. Then you have to indicate which domains will be managed by happyDomain.

### Adding a shared secret {#shared-secret}

Under the main [`key`](https://knot.readthedocs.io/en/latest/reference.html#key-section) section of your configuration, add the following key:

```yaml
key:
  [...]
  - id: happydomain
    algorithm: hmac-sha512
    secret: "<SOME_SECRET>"
```

Obviously replace `<SOME_SECRET>` with a string as obtained with `openssl rand -base64 48`.


### Creating an authorization rule for happyDomain

In addition to the key, you must specify in the configuration how the key can be used.

To do this, under the main [`acl`](https://knot.readthedocs.io/en/latest/reference.html#acl-section) section, we add:

```yaml
acl:
  [...]
  - id: acl_happydomain
    key: happydomain
    action: transfer
    action: update
```

This associates the `key` defined just before with the actions `transfer` and `update`, respectively to allow retrieving the zone and to update records.


### Associate the authorization to each zone

Now that you have created a rule allowing the `happydomain` key to make changes, you need to indicate to which zones this rule applies.

For each [zone](https://knot.readthedocs.io/en/latest/reference.html#zone-section), you must add an [`acl`](https://knot.readthedocs.io/en/latest/reference.html#acl) element referencing the `acl_happydomain` rule:

For example, for an existing `happydomain.org` zone, we will add the `acl` line as follows:

```yaml
zone:
  [...]
  - domain: happydomain.org
    acl:
      - acl_happydomain
    [...]
```

The `acl` element is a list, so you may already have other acl elements in this list. In this case you just need to add the `acl_happydomain` element to the already existing list.

You have to add this `acl` element for each zone, unless you use the following trick.


### Associate the authorization to all zones

If you manage many zones, it may be more convenient to set the default authorization for all zones. In this case, instead of the previous section, we will modify the `default` template:

```yaml
template:
  - id: default
    acl:
      - acl_happydomain
  [...]
```

The `default` template is applied to all zones by default. By doing so, all zones will inherit the `acl_happydomain` rule.


### Apply the configuration

Now that the configuration file has been modified, tell `knotd` to reload its configuration:

```sh
knotc reload
```


## Link happyDomain and knot


Once `knot` well configured, you can link it to happyDomain using [the *Dynamic DNS* connector]({{% ref "/pages/provider-new-choice.md" %}}) :

![The Dynamic DNS connector on the host selection page](/img/choose-dynamic-dns.png)

Then fill in the form with the address where your `knot` server is accessible, then fill in the different *Key* fields with [the information from the `knot`'s `key` section](#shared-secret):

- **Key Name** : corresponds to `id` in knot's configuration ;
- **Key Algorithm** : corresponds to `algorithm` ;
- **Secret Key** : corresponds to `secret`.

Once the provider is added, it does not allow you to list existing domains, but you can still manually add all your domains.
