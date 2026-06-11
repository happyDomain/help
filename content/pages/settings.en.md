---
date: 2026-06-10T12:00:00+02:00
title: Your account and settings
author: nemunaire
weight: 2000
description: "Manage your happyDomain account and tailor the interface: language, field hints, confirmation, zone view, password and account deletion"
aliases:
    me
---

happyDomain lets you manage your account and tailor how the interface behaves. Your preferences are saved with your account, so you find them again on every device you connect from.

You reach this page from the **My Account** link in the top menu. Each option is described below.

<!-- TODO: screenshot of the settings form -->

## Language

Choose the language used throughout the interface. The list contains every language currently available in your happyDomain instance. Changing it takes effect as soon as you save.

## Field hints

Form fields often come with a short help text. This setting controls how that help is displayed:

- **Hide** -- no help text is shown.
- **Tooltip near field** -- the help appears as a tooltip when you hover the field.
- **Under field when focused** -- the help appears below the field, but only while you are editing it.
- **Under field, always** -- the help is permanently displayed below each field.

Pick the level of guidance that suits your familiarity with DNS.

## Confirmation before applying

When you publish changes to your DNS provider, happyDomain can show a confirmation step beforehand. This setting decides when:

- **Ask on unexpected differences** -- the confirmation appears only when the changes differ from what was expected.
- **Always ask** -- a confirmation is always shown before applying.
- **Never ask, overwrite on publish** -- changes are applied directly, without a confirmation step.

See [Publishing changes]({{% relref "publish-changes" %}}) for the full publication workflow.

## Zone view layout

Choose how a domain's zone is displayed:

- **Grid view (easiest)** -- the most visual and approachable layout.
- **List view (fastest)** -- a compact list, quicker to scan.
- **List with records (advanced)** -- a list that also exposes the underlying DNS records, for advanced users.

## Show DNS record types

This switch shows the resource record type (A, MX, CNAME, etc.) associated with each service. It is meant for users who are already familiar with DNS and want to see the technical record types behind the friendly service names.

## Newsletter

The option to receive news about future improvements is proposed when you [create your account]({{% relref "signup" %}}). To change your subscription afterwards, refer to the unsubscribe link included in the messages you receive.

## Saving

When your preferences are set, click **Save settings**. A confirmation message appears and the new settings take effect immediately, including a language change if you made one.

## Change password

A dedicated part of the page lets you change the password of your account.

## Change account email address

It is currently not possible to change the email address of your account. We invite you to contact us if you wish to change it.

## Delete your account

The last part of the page allows you to delete your happyDomain account.

Once the deletion is validated, your account will no longer be accessible and all data belonging to you will be deleted shortly after, during a regular database cleanup.

From the moment you delete your account, your domains will continue to respond according to the last update you made on happyDomain. The deletion will not affect the distributed data.
