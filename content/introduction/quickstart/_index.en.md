---
date: 2025-06-15T11:44:00+02:00
title: Quickstart
author: nemunaire
weight: 1
---

happyDomain is a service that centralizes the management of your domain names from different registrars, hosts or authoritative DNS servers.
It's a web interface and a REST API that offer a simpler domain experience than most of the interfaces we usually see for managing domains, including features we'd expect to see in 2025.

At happyDomain, we want to make sure that domain names and DNS are no longer a daunting experience, but instead give you all the tools you need to understand and make changes in peace.

Can't wait to get started? Follow the guide!


## 1. Online or on premise

happyDomain is free (as in free speech) software.
This means, among other things, that you can install it at home.

If you're familiar with Docker, you can follow the [dedicated installation guide]({{% relref "../deploy/docker" %}}).

If you're not familiar with the command line, or if you'd like to evaluate the software quickly, we recommend that you create an account on our online service.

Go to : <https://app.happydomain.org/join>


## 2. Add a domain to manage

happyDomain will connect to your hosting provider (or local authoritative server).
Your domain remains hosted where it is today; using happyDomain does not imply any transfer or change of ownership.

{{% notice style="info" title="I don't have a domain name yet" icon="question" %}}

We don't sell domain names, you must already have one [with a supported hosting provider](https://app.happydomain.org/providers/features).

{{% /notice %}}

When you log on to happyDomain for the first time, a wizard will guide you through the process of linking your first domain.

Depending on your hosting provider, the procedure will differ.
But for most, you'll need to go to your host's customer account, and request an API key.

For OVH, the procedure is simplified, as all you have to do is follow the instructions and authorize happyDomain to access the domain-related part of your account.

If you have your own authoritative server, you'll need to get the keys to interact with it either from the administrator or by looking in the configuration.

Once the connection between happyDomain and your first host or server has been established, all you have to do is select the domains you want happyDomain to manage.


## 3. Consult the DNS zone

The DNS zone refers to the technical content of your domain.

To view the zone corresponding to a domain, click on the domain name that appears on the happyDomain home page.

After a few seconds of fully automatic import and analysis, you'll immediately see a list of registrations or services as they are currently distributed to your visitors.

{{% notice style="primary" title="About the “services”" %}}

The complexity of DNS stems in part from the mismatch between purely technical constraints and the actual use of records.

happyDomain tries to simplify this by grouping technical records under their concrete uses. This is what we call “service”.

{{% /notice %}}


## 4. Modify a record

In the zone display screen, click on a record to view its details.

Each DNS record has particular characteristics and constraints.
Context-sensitive help tries as far as possible to give you the essential information to guide you through the modifications you need to make.

When you make a modification to a record, it is not directly published to your host or server.


## 5. Distribute your changes

Once you've made all the changes you need, click on the “Distribute my changes” button.

A dialog box will appear showing you the exact changes that will be applied to your host.
At this point, there's still time to select the changes you don't want/no longer want to be applied.

Before validating the window, you can add a message that will be recorded in the log, enabling you to quickly recall the reason for the change.

After all, one of the advantages of happyDomain is that you can easily go back and undo a change if you see a mistake.
[History is the subject of a dedicated help page.]({{% relref "../../pages/domain-history" %}})

---

So now you know how to use happyDomain's main features.
If you encounter any problems or have any ideas for improvement, [please let us know](https://github.com/happyDomain/happyDomain/issues/new).
