---
date: 2026-06-10T12:00:00+02:00
title: Notifications
author: nemunaire
weight: 2300
description: "Receive alerts when your monitoring checks change state, and configure where and when those alerts are delivered"
---

happyDomain can notify you when something changes on your domains. Notifications are driven by the [monitoring & checks]({{% relref "checks" %}}) system: whenever a checker changes status (for example from **OK** to **Warning**, or back to **OK** after a problem), happyDomain can deliver an alert to the channels you have configured.

You manage notifications from the **Notifications** page in your account settings. It is organised into three tabs: **Channels**, **Preferences** and **History**.

<!-- TODO: screenshot of the notifications page with its three tabs -->


## What triggers a notification

Notifications are tied to your checkers. happyDomain watches the status reported by each check and sends a notification when:

- a check **degrades** to (or beyond) a severity you care about, for example reaching **Warning**, **Critical** or **Error**;
- a check **recovers**, returning to a healthy state, if you have asked to be notified of recoveries.

Each notification therefore describes a status transition (the previous status and the new status) for a given target. Which transitions reach you, and through which channels, is entirely controlled by your **preferences** (see below).


## Channels

A **channel** is a destination where notifications are sent. Open the **Channels** tab to manage them.

To add one, click **Add**, choose a **Type**, give it a **Name** (so you can recognise it later) and fill in the type-specific fields. A channel can be **enabled** or disabled with a switch without deleting it.

happyDomain offers the following channel types out of the box:

### Email

Sends notifications to an email address.

- **Email address**: the recipient. If left empty, the notification is sent to your account's email address.

### Webhook

Sends an HTTP request to a URL of your choice, which is useful to integrate happyDomain with chat tools, automation platforms or your own services.

- **Webhook URL** (required): the endpoint that will receive the notification.
- **Webhook headers**: optional custom HTTP headers (name/value pairs) to add to the request, for example an authorization header.
- **Webhook secret**: an optional secret used to sign the request so the receiver can verify it really comes from happyDomain.

### UnifiedPush

Delivers push notifications to your devices through a [UnifiedPush](https://unifiedpush.org/) distributor.

- **UnifiedPush endpoint** (required): the endpoint URL provided by your UnifiedPush application.

{{% notice style="info" title="Other channel types" icon="circle-info" %}}
The list of available types depends on what the instance administrator has enabled. For types that happyDomain does not provide a dedicated form for, the editor falls back to a raw JSON configuration field.
{{% /notice %}}

### Testing a channel

Once a channel is created and enabled, use the **send/test** button next to it to deliver a test notification. This confirms the configuration works before relying on it for real alerts.

<!-- TODO: screenshot of the channels list with the test button -->


## Preferences

Channels say *where* notifications go; **preferences** say *what* gets sent and *when*. Open the **Preferences** tab and click **Add** to create a rule.

A preference combines the following settings:

### Scope

Choose how broadly the rule applies:

- **Global**: applies to all your domains and services.
- **Domain**: applies to a single domain you select.
- **Service**: applies to a specific service (you select the domain and provide the service identifier).

### Channels

Select one or more of your configured channels to receive the notifications matching this preference. If you have no channels yet, create one first in the **Channels** tab.

### Minimum status

Pick the lowest severity that should trigger a notification. The available levels, in increasing severity, are **OK**, **Info**, **Warning**, **Critical** and **Error**. Only status changes that reach this level (or higher) are notified. For example, choosing **Warning** means you are alerted on Warning, Critical and Error, but not on purely informational changes.

### Notify on recovery

When enabled, you also receive a notification when a previously degraded check returns to a healthy state, so you know when a problem has been resolved.

### Quiet hours

Optionally define a period during which notifications are held back, for instance overnight. Set a **start** hour and an **end** hour (0 to 23). When quiet hours are active, alerts raised within that window are not sent immediately.

### Enabled

Each preference can be turned on or off with a switch, letting you temporarily suspend a rule without deleting it.

<!-- TODO: screenshot of the preference editor -->

{{% notice style="tip" title="Start simple" icon="lightbulb" %}}
A common setup is a single **Global** preference, pointing at one channel, with a minimum status of **Warning** and recovery notifications enabled. You can later add more specific per-domain or per-service rules as your needs grow.
{{% /notice %}}


## History

The **History** tab lists the notifications happyDomain has attempted to send. For each entry you can see:

- the target concerned;
- the status transition (previous status → new status);
- whether delivery **succeeded** or **failed** (with the error message when it failed).

Use **Load more** to page back through older entries. This view is the place to check why an expected alert did not reach you, for example a channel misconfiguration causing repeated failures.

<!-- TODO: screenshot of the notification history list -->
