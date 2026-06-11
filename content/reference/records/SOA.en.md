---
date: 2021-01-12T21:38:49+02:00
author: Frederic
title: SOA (Start Of Authority)
description: "Understand the Start Of Authority record, the mandatory record at the apex of every DNS zone, and how happyDomain manages it for you."
weight: 10
aliases:
    records/SOA
---

The **Start Of Authority** (SOA) record is defined by [RFC 1035](https://www.rfc-editor.org/rfc/rfc1035). It is mandatory and unique: exactly one SOA must sit at the apex (the root) of every DNS zone. Its presence declares that the name server is authoritative for the zone and carries the parameters that govern how the zone is replicated between servers and cached by resolvers.

## Fields of the SOA record

The SOA record gathers seven values:

| Field | Description |
|---|---|
| **MNAME** (primary name server) | The hostname of the primary (master) name server for the zone. |
| **RNAME** (responsible party) | The email address of the person responsible for the zone, encoded in DNS form: the `@` is replaced by a dot (for example `hostmaster.example.com.` means `hostmaster@example.com`). |
| **Serial** | A version number for the zone. It must increase every time the zone changes, so that secondary servers know they need to transfer the new content. |
| **Refresh** | How often (in seconds) a secondary server checks the primary for an updated serial. |
| **Retry** | How long (in seconds) a secondary waits before retrying a failed refresh. |
| **Expire** | How long (in seconds) a secondary keeps serving the zone when it cannot reach the primary, before considering the data stale. |
| **Minimum** (negative-cache TTL) | The duration for which resolvers cache negative answers (NXDOMAIN), per [RFC 2308](https://www.rfc-editor.org/rfc/rfc2308). |

## The SOA in happyDomain

happyDomain does not present the SOA as a standalone record to edit field by field. Instead, the apex of your zone is modelled as an **Origin** service, which groups the SOA together with the zone's name servers (NS records). You will therefore find the SOA at the root of your domain, alongside the list of authoritative name servers, rather than in a separate form.

The **serial** is, in most cases, handled automatically. When you publish your changes, many DNS providers manage the serial themselves: happyDomain detects this capability and re-reads the zone after publication to reflect the serial the provider actually assigned. You normally do not need (nor are you expected) to set it by hand.

{{% notice style="info" title="What you can and cannot change" %}}
The exact behaviour depends on your DNS provider. Some providers expose the full SOA and let happyDomain submit its values; others manage the SOA serial — and sometimes the other timers — on their side. When the provider takes care of the serial, any value happyDomain shows for it simply reflects the published state and is refreshed automatically.
{{% /notice %}}

For more on how the apex and other groupings are represented, see the {{< relref "../services" >}} chapter.
