---
date: 2021-01-12T21:38:49+02:00
title: Features
author: Frederic
archetype: chapter
weight: 10
---

This section walks through happyDomain feature by feature, following the path a new user
takes: from creating an account to publishing a fully managed zone.

You start by [creating an account]({{% relref "signup" %}}) and [logging in]({{% relref "login" %}}),
then land on the [dashboard]({{% relref "domains" %}}) that centralizes all your domains.

Before importing a domain, you connect a [provider]({{% relref "provider-list" %}}): happyDomain
[adds]({{% relref "provider-new-choice" %}}) and [configures]({{% relref "provider-update" %}})
it for you, whatever its [specific features]({{% relref "provider-features" %}}).

You can then [import a domain]({{% relref "domain-new" %}}) and explore it in the
[zone editor]({{% relref "domain-abstract" %}}), which
lets you manage [subdomains]({{% relref "subdomains" %}}) and group records into
[services]({{% relref "services" %}}). Every change is reviewed
before you [publish]({{% relref "publish-changes" %}}) it, and the
[history]({{% relref "domain-history" %}}) keeps track of everything. You can also
[import or export]({{% relref "import-export" %}}) the whole zone as a standard file.

happyDomain also helps you keep an eye on your domains with
[monitoring and checks]({{% relref "checks" %}}), [availability lookups]({{% relref "domain-availability" %}})
and the [domain test tool]({{% relref "tools-client" %}}).

Finally, you can fine-tune your account: [your account and settings]({{% relref "settings" %}}),
[sessions and API access]({{% relref "api-keys" %}}),
[notifications]({{% relref "notifications" %}}) and [quotas]({{% relref "quotas" %}}).

{{% children %}}
