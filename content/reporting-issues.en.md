---
date: 2026-08-12T10:00:00+02:00
title: Reporting an issue
author: nemunaire
weight: 30
---

**Tell us in one sentence, that's enough.** We'll come back to you if we need more.

Given the diversity of DNS configurations and providers, we can't test every possible setup: your reports are how happyDomain gets better for everyone.
Don't spend an evening investigating before writing to us, and don't worry about reporting something that turns out not to be a bug: we'd much rather read a report that's too short than never hear about the problem.

Where to report:

- from happyDomain itself: **Report a problem**, in the user menu, or directly on an unexpected error message, where the failing error travels with your report. happyDomain fills in the technical details for you, you just describe what you were doing;
- directly on the forge you prefer, we read them all: [GitHub](https://github.com/happyDomain/happydomain/issues), [GitLab](https://gitlab.com/happyDomain/happydomain/-/issues), [Framagit](https://framagit.org/happyDomain/happydomain/-/issues) or [Codeberg](https://codeberg.org/happyDomain/happyDomain/issues);
- by email at [contact@happydomain.org](mailto:contact@happydomain.org), if you have no account on a forge (the **Report a problem** dialog can prepare that email for you);
- on the [Matrix channel](https://matrix.to/#/#happyDNS:matrix.org), if you'd rather just talk about it;
- **for security vulnerabilities only**, privately: see [our security policy](https://github.com/happyDomain/happydomain/blob/master/SECURITY.md).


## The *Report a problem* button

happyDomain records the errors it runs into as you use it.
When you open **Report a problem**, it prepares a report containing your version, your browser and those errors, then opens a pre-filled issue on the forge of your choice, or copies everything to your clipboard if you'd rather paste it somewhere else.

You can read and edit those details before sending them: they contain no password, no API key and no domain name.


## If you want to go further

Everything below is optional, and we'll ask for it only if the report needs it.

### happyDomain's logs

Many bugs leave a trace on the server side, and an error such as `NetworkError` in your browser is often a crash in happyDomain.
The full text of such a crash tells us exactly which line broke.

With Docker Compose, from the directory holding your `docker-compose.yml`:

```
docker compose logs -f happydomain
```

With a plain container:

```
docker container logs -f happydomain
```

With the standalone binary, the logs are written to the terminal running it (or captured by your service manager: `journalctl -u happydomain`).

Reproduce the problem while the logs are being displayed, then copy what appeared.
Check for API keys or passwords before pasting them.

### Your browser's errors

1. open the developer tools with `F12` (or `Ctrl+Shift+I`, `Cmd+Option+I` on macOS);
2. select the **Console** tab, and clear it;
3. reproduce the problem;
4. copy the messages that appeared, and, in the **Network** tab, the failing request.

A screenshot works too. Copied text is easier for us to search and quote back to you, but never let that stop you from reporting.


## Checking a fix

When we tell you a fix has landed, update your instance before testing again.

With Docker Compose:

```
docker compose up -d --pull always
```

With a plain container, pull the image then recreate the container:

```
docker image pull happydomain/happydomain
```

Fixes land on the `master` image and on <https://get.happydomain.org/master/> as soon as they're merged, before the next release.
If the problem is still there, tell us: we'd rather hear about it twice than leave it broken.
