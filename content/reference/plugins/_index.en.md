---
date: 2025-06-15T11:38:46+02:00
title: Plugins
author: nemunaire
archetype: chapter
weight: 40
---

Plugins are external pieces of code — shared libraries loaded at startup — that extend happyDomain's functionality without recompiling the server. An operator simply drops a `.so` file into a configured directory, and happyDomain picks it up automatically on the next start.

{{% children %}}
