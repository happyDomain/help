---
date: 2026-06-10T12:00:00+02:00
title: Quotas & limits
author: nemunaire
weight: 2400
description: "Understand the limits an administrator may set on your account, and what happens when one is reached"
---

A happyDomain instance can apply **quotas** to each account. Quotas are limits set by the person running the instance (the administrator), not by you. They exist mainly to keep a shared instance fair and to avoid overloading the servers, and they apply above all to the [monitoring & checks]({{% relref "checks" %}}) system.

{{% notice style="info" title="Quotas are managed by the administrator" icon="circle-info" %}}
Quotas are **read-only** for regular users. There is no screen in your account where you can view or change your own quota: they are configured server-side and adjusted only through the administration interface. If a limit is getting in your way, the right course of action is to contact the administrator of your instance.
{{% /notice %}}


## What quotas can cover

On a default happyDomain instance, quotas relate to how the monitoring scheduler works for your account. An administrator may set, per account or instance-wide:

- **Maximum checks per day**: a cap on how many checker executions are run automatically for you each day. Once the cap is reached, further scheduled checks wait until the next day.
- **Result retention**: how long the results of your checks are kept before old executions are automatically cleaned up.
- **Inactivity pause**: after a period without logging in, the scheduler may stop running your automatic checks until you sign in again. This keeps the instance from spending resources on abandoned accounts.
- **Scheduling pause**: an administrator can entirely pause automatic scheduling for an account.

Each of these can be left at the instance default, set to a specific value for your account, or explicitly made unlimited, at the administrator's discretion.

{{% notice style="tip" title="No fixed limit on the number of domains" icon="lightbulb" %}}
On a standard happyDomain instance, the quota system does not impose a built-in cap on the number of domains you can manage. If a particular instance does restrict this, it is a policy of that instance; ask its administrator for the details that apply to you.
{{% /notice %}}


## What you see when a limit is reached

Quotas mostly act in the background. You will not usually see a quota screen; instead you notice their effects:

- **Automatic checks stop running**: if your daily check cap is reached, or your account is in an inactivity or scheduling pause, scheduled checks simply do not run until the condition clears. You can still trigger a check **manually** from the [checks]({{% relref "checks#running-a-check-manually" %}}) interface.
- **Older results disappear**: with a retention limit, executions older than the allowed age are removed during routine cleanup, so the history only goes back so far.
- **An action is refused**: where an operation would exceed a limit, the interface shows an error message explaining what went wrong.

If you are unsure whether a behaviour is caused by a quota, or you need a limit raised, contact the administrator of your instance.
