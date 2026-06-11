---
date: 2026-06-10T12:00:00+02:00
title: Sessions & API access
author: nemunaire
weight: 2200
description: "Manage your active sessions and use authentication tokens to call the happyDomain REST API"
---

happyDomain does not have a separate "API keys" feature. Instead, every way of accessing the platform, whether through the web interface or the REST API, relies on the same kind of **session token**. Managing your sessions therefore also means managing your API access.

You manage your sessions from the security section of your account, under **Active Sessions**.

<!-- TODO: screenshot of the sessions manager -->

## Listing your active sessions

The list shows every session currently open on your account. For each one you see:

- its **description** (the name given when it was created), followed by a short fingerprint derived from the token;
- a **current session** badge on the session you are using right now;
- when it was **created**, when it was **last used**, and when it **expires**.

Sessions are sorted with the most recently used first.

## Creating a token for API access

To obtain a token you can use to call the API:

1. Click **Create API key**.
2. Give the session a **description** (for example the name of the script or machine that will use it). This helps you recognise it later in the list.
3. Validate the creation.

The token secret is then displayed **once**. Use the eye button to reveal it and the copy button to put it on your clipboard.

{{% notice style="warning" title="Copy the secret immediately" icon="triangle-exclamation" %}}
The session secret is shown only at creation time and is never displayed again. Copy it and store it somewhere safe before closing the dialog. If you lose it, delete the session and create a new one.
{{% /notice %}}

## Using a token with the REST API

The token is passed in the HTTP `Authorization` header as a Bearer token. For example, to list your domains:

```sh
curl -H "Authorization: Bearer YOUR_SECRET_TOKEN" https://your-happydomain-server/api/domains
```

Replace `YOUR_SECRET_TOKEN` with the secret you copied, and the host with the address of your happyDomain instance. Any API endpoint is reached the same way, by sending this header with each request.

## Revoking a session

To revoke a session (and the token associated with it), click the delete button on its row. The token stops working immediately. You cannot delete the session you are currently using from this list; to end it, simply log out, or use the option below.

{{% notice style="info" title="Terminate all sessions" icon="circle-info" %}}
The **Terminate all sessions** button closes every session at once, including the one you are using. This is useful if you suspect a token has leaked: all existing tokens become invalid and you are sent back to the [login page]({{% relref "login" %}}). You will then need to log in again and recreate any API tokens you still need.
{{% /notice %}}
