---
date: 2026-06-11T09:00:00+02:00
author: nemunaire
title: Outbound deliverability (happyDeliver)
description: "Send a real test message from your domain to a happyDeliver instance and grade how well it would be delivered to a real inbox."
weight: 210
---

The **Outbound deliverability** checker (named *Outbound deliverability (via happyDeliver)* in happyDomain) measures how a message sent **from your domain** would actually fare on its way to a recipient's inbox. Rather than inspecting DNS records in isolation, it drives an external [happyDeliver](https://git.nemunai.re/happyDomain/happyDeliver) instance to perform an end-to-end test.

This checker is **service-level**: it is attached to a service (typically the mail configuration of a subdomain) and needs SMTP credentials to send the test message on your behalf.

## How it works

For each run, the checker:

1. Allocates a fresh recipient address on the configured happyDeliver instance.
2. Sends a **real test email** from the tested domain to that address, using the SMTP submission server and credentials you provide.
3. Polls happyDeliver until the message has been received and analysed.
4. Stores happyDeliver's report and exposes one score per section as a metric.

happyDeliver grades the message across several sections (DNS, authentication, spam filters, blacklists, headers, content) and computes an overall score. The checker turns each section into a rule.

{{% notice style="info" title="An external happyDeliver instance is required" %}}
This checker does nothing on its own: it needs a reachable happyDeliver instance, identified by its API URL and bearer token. Those are usually configured once by the administrator and can be overridden per domain.
{{% /notice %}}

## What it checks

Each section rule compares happyDeliver's score for that section against a configurable minimum. The check is **Critical** when the score falls below the minimum, otherwise **OK**.

| Rule | What it verifies | Default minimum |
|---|---|---|
| `happydeliver.score.overall` | happyDeliver's Overall score | 70 |
| `happydeliver.score.dns` | DNS configuration score | 70 |
| `happydeliver.score.authentication` | Authentication score (SPF / DKIM / DMARC) | 80 |
| `happydeliver.score.spam` | Spam-filter score | 70 |
| `happydeliver.score.blacklist` | Blacklist score | 90 |
| `happydeliver.score.header` | Header score | 70 |
| `happydeliver.score.content` | Content score | 60 |

A separate `happydeliver.lifecycle` rule reports the outcome of the run itself: **OK** when the message was analysed, **Critical** when the test address could not be allocated, the message could not be sent, or happyDeliver returned a wait/fetch/parse error, and **Warning** when the message was not analysed before the timeout elapsed.

Each section minimum can be tuned through its own `min_score_<section>` rule option in the happyDomain interface.

## Options

### Sending (per domain)

| Option | Meaning | Default |
|---|---|---|
| Sending SMTP host (`smtp_host`) | Hostname or IP of the submission server used to send the test email. **Required.** | (none) |
| Sending SMTP port (`smtp_port`) | Submission port (587 for STARTTLS, 465 for implicit TLS, 25 for plain). | 587 |
| SMTP username (`smtp_username`) | Username for the submission server (omit for anonymous submission). | (none) |
| SMTP password (`smtp_password`) | Password for the submission server. | (none) |
| TLS mode (`smtp_tls`) | How to negotiate TLS: `starttls`, `tls`, or `none`. | `starttls` |
| From address (`from_address`) | Address used in the `From` header; must belong to the tested domain. | `no-reply@<domain>` |

### Message content (per domain)

| Option | Meaning | Default |
|---|---|---|
| Subject (`subject_override`) | Override the default test subject. | (built-in) |
| Plain-text body (`body_text_override`) | Override the default plain-text body. | (built-in) |
| HTML body (`body_html_override`) | Override the default HTML body. | (built-in) |

### Timing (per domain)

| Option | Meaning | Default |
|---|---|---|
| Wait timeout (`wait_timeout`) | Seconds to wait for happyDeliver to receive and analyse the message. | 900 |
| Poll interval (`poll_interval`) | Seconds between status polls (clamped to the 2–60 range). | 5 |

### Instance (admin, overridable per domain)

| Option | Meaning | Default |
|---|---|---|
| happyDeliver instance URL (`happydeliver_url`) | Base URL of the happyDeliver API. | (admin) |
| happyDeliver API token (`happydeliver_token`) | Bearer token for the happyDeliver API. | (admin) |

## In happyDomain

Enable this checker from the service's **Checks** tab and provide the SMTP submission details so happyDomain can send the test message. See {{< relref "/pages/checks" >}} for the general workflow of scheduling and reading checks.

Because deliverability depends heavily on your anti-spoofing posture, pair this checker with a well-configured {{< relref "/reference/services/email" >}} setup (SPF, DKIM and DMARC). For the DNSSEC half of your domain's trust chain, see {{< relref "/reference/checkers/dnssec" >}}.
