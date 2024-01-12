---
date: 2021-01-12T21:38:49+02:00
title: The Home page
author: Frederic
aliases:
    domains
weight: 10
---

## Your domains

The home page presents the list of all the domains managed by happyDomain, whatever their host:

The domains managed by happyDomain](domain-list.png)

Click one of the domains to start [make changes]({{% relref "domain-abstract" %}}) (add a sub-domain, add a service, ...).


## Your registries and domain hosts

On the right, you can see the list of the different hosting providers for your:

The hosters of your domains](hosters-list.png)

You can [add new host]({{% relref "source-new-choice" %}}) by clicking on the + button in the table header.

Clicking on a row in this table will filter the list of domains to show only domains managed by this host.

You will also see, if the host allows you to list the domains that belong to you, the domains that you can add to happyDomain:

![Domain filtering according to the host](hoster-ovh.png)

To view the entire list again, simply click on the selected host again.


### Modify or remove a host

If you find an error or no longer need a hosting provider, click on the ... on the line of the host concerned. You will then be able to choose between [update information]({{% relref "source-update" %}}) or delete the host:

modify or delete a host](hoster-edit.png)

Note that you will not be able to remove the host as long as domains referring to it exist in the list on the left.


## Add a domain

You have a new domain you want to manage in happyDomain? Start by entering its name in the field below the list. You will then be guided to the [to choose the host] screen ({{% relref "domain-new" %}}).

Location to add a domain that is not listed](new-domain.png)

The field does not show when a host is selected on the right. Unless this host does not allow to list:

![Special case of addition for authoritative DNS servers](hoster-self.png)

In this case, validating the field will automatically search for the new domain with the selected host, as indicated by the message just above the field.
