---
title: "Writing a happyDomain Checker Plugin"
description: "Technical guide for developing checker plugins for happyDomain"
---

happyDomain can be extended with external **checker plugins** — shared libraries (`.so` files) that add automated diagnostics on zones, domains, services or users. A checker plugin is loaded into the running happyDomain process at startup; the operator simply drops a `.so` file into a configured directory, no recompilation of the server required.

A checker has two halves: it **collects** raw data about a target (an observation), then **evaluates** that data against a set of rules to produce a status. Results are stored and displayed in the happyDomain UI alongside the domain or service they concern.

{{< notice style="warning" >}}
A `.so` plugin is loaded into the happyDomain process and runs with the same privileges as the server. Treat the plugin directory as a trusted location: happyDomain refuses to load plugins from a directory it cannot trust (see [Security and deployment](#security-and-deployment)).
{{< /notice >}}

---

## What a checker plugin must export

Plugins are built against the **`checker-sdk-go`** module, published separately from the happyDomain core. Throughout this page, `checker` refers to the package `git.happydns.org/checker-sdk-go/checker`.

happyDomain's loader looks for a single exported symbol named `NewCheckerPlugin` with this exact signature:

```go
func NewCheckerPlugin() (*checker.CheckerDefinition, checker.ObservationProvider, error)
```

The two return values describe the two halves of a checker:

- **`*CheckerDefinition`** describes the checker: its identifier, name, version, the observation keys it relies on, the options it accepts, its rules, an optional aggregator, a scheduling interval, and whether it exposes HTML reports or metrics. See the [field table](#checkerdefinition-fields) below.
- **`ObservationProvider`** is the data-collection half. It exposes a `Key()` (the observation key the rules look up) and a `Collect(ctx, opts)` method that returns the raw observation payload. happyDomain serialises that result to JSON and caches it per observation context.
- Return a non-nil `error` if the plugin cannot initialise (a missing environment variable, a broken cgo dependency, …). The host logs the error and skips the file rather than aborting startup.

A single `.so` may export several plugin kinds. The loader runs every known plugin loader against every file, then skips any symbol it does not recognise, so one binary can ship more than one plugin.

---

## Minimal example

This is the smallest plugin that loads. It collects a fixed observation and declares no rules. Adapt it from [`checker-dummy`](https://git.happydns.org/checker-dummy), the reference implementation.

```go
// Command plugin is the happyDomain plugin entrypoint for the dummy checker.
//
// Build with:
//   go build -buildmode=plugin -o checker-dummy.so ./plugin
package main

import (
	"context"

	"git.happydns.org/checker-sdk-go/checker"
)

type dummyProvider struct{}

func (dummyProvider) Key() checker.ObservationKey { return "dummy.observation" }

func (dummyProvider) Collect(ctx context.Context, opts checker.CheckerOptions) (any, error) {
	return map[string]string{"hello": "world"}, nil
}

// NewCheckerPlugin is the symbol resolved by happyDomain at startup.
func NewCheckerPlugin() (*checker.CheckerDefinition, checker.ObservationProvider, error) {
	def := &checker.CheckerDefinition{
		ID:              "com.example.dummy",
		Name:            "Dummy checker",
		Version:         "0.1.0",
		ObservationKeys: []checker.ObservationKey{"dummy.observation"},
		// Add Rules / Aggregator / Options here in a real plugin.
	}
	return def, dummyProvider{}, nil
}
```

{{< notice style="warning" >}}
A Go plugin and the host process share the same runtime. They **must** be compiled with the same Go toolchain version and the same versions of every shared dependency. Any mismatch produces a hard error at load time. See [Build constraints](#build-constraints).
{{< /notice >}}

---

## `CheckerDefinition` fields

The `*CheckerDefinition` returned by `NewCheckerPlugin` is the description of your checker:

| Field | Type | Description |
|---|---|---|
| `ID` | `string` | **Required.** Stable, persistent identifier. Pick a namespaced value (`com.example.dnssec-freshness`, not `dnssec`) and never change it: it keys stored results and user configuration. |
| `Name` | `string` | Human-readable name shown in the UI. |
| `Version` | `string` | Plugin version (e.g. `"1.0.0"`). |
| `Availability` | `CheckerAvailability` | Declares which scopes the checker applies to and any provider/service restrictions (see below). |
| `Options` | `CheckerOptionsDocumentation` | Documents the options the checker accepts, grouped by scope (see below). |
| `Rules` | `[]CheckRule` | The rules evaluated against the collected observation. |
| `Aggregator` | `CheckAggregator` | Optional. Combines the per-rule `CheckState`s into a single summary state. |
| `Interval` | `*CheckIntervalSpec` | Optional scheduling bounds (`Min`, `Max`, `Default` durations). |
| `HasHTMLReport` | `bool` | Set when the provider implements `CheckerHTMLReporter`. |
| `HasMetrics` | `bool` | Set when the provider implements `CheckerMetricsReporter`. |
| `ObservationKeys` | `[]ObservationKey` | The observation keys this checker reads. |

### Availability

`CheckerAvailability` controls where the checker is offered in the UI:

| Field | Type | Description |
|---|---|---|
| `ApplyToDomain` | `bool` | Checker can run against a whole domain. |
| `ApplyToZone` | `bool` | Checker can run against a zone. |
| `ApplyToService` | `bool` | Checker can run against a specific service. |
| `LimitToProviders` | `[]string` | Restrict to certain DNS provider identifiers (empty = no restriction). |
| `LimitToServices` | `[]string` | Restrict to certain service type identifiers, e.g. `"abstract.MatrixIM"` (empty = no restriction). |

### Options

Options are declared grouped by **scope**, i.e. who sets them and how long they persist. Each scope is a slice of `CheckerOptionDocumentation`:

| Scope | Who sets it | Typical use |
|---|---|---|
| `AdminOpts` | Administrator | Instance-wide settings, shared credentials. |
| `UserOpts` | User | Personal preferences (e.g. language). |
| `DomainOpts` | User | Domain-level configuration. |
| `ServiceOpts` | User | Service-level configuration. |
| `RunOpts` | User, at run time | Per-invocation parameters. |

happyDomain merges the scoped values from least specific (admin) to most specific (run-time) before calling `Collect`, so the provider receives a single flat `CheckerOptions` map. Each option is a `CheckerOptionField` with fields such as `Id`, `Type`, `Label`, `Default`, `Choices`, `Required`, `Secret`, `Description` and `AutoFill`. Read typed values out of the map with the SDK helpers `checker.GetOption`, `checker.GetIntOption`, `checker.GetBoolOption`, …

{{< notice style="info" >}}
When happyDomain registers an externalisable checker, it automatically appends an `endpoint` admin option, so the administrator can delegate collection to a remote HTTP endpoint instead of running the checker in-process. Leave it empty to run locally.
{{< /notice >}}

---

## The `ObservationProvider`

The provider is the data-collection half of the checker:

```go
type ObservationProvider interface {
	Key() ObservationKey
	Collect(ctx context.Context, opts CheckerOptions) (any, error)
}
```

- `Key()` returns the observation key this provider fills. It must match one of the `ObservationKeys` declared in the definition.
- `Collect` performs the actual work (a DNS query, an HTTP call, …) and returns any JSON-serialisable value. happyDomain marshals it to JSON and caches it; the rules then read it back.

A provider can optionally implement additional SDK interfaces to extend its behaviour:

| Interface | Purpose |
|---|---|
| `CheckerHTMLReporter` | `GetHTMLReport(ctx ReportContext)` renders the stored observation as an HTML document. |
| `CheckerMetricsReporter` | `ExtractMetrics(ctx ReportContext, collectedAt)` produces time-series metrics. |
| `CheckEnabler` | `IsEligible(ctx, opts)` decides, from the target's actual data, whether running the checker is meaningful at all. |
| `DiscoveryPublisher` | `DiscoverEntries(data)` publishes `DiscoveryEntry` records other checkers can consume. |

Rules return `CheckState` values whose `Status` is one of `StatusOK`, `StatusInfo`, `StatusWarn`, `StatusCrit`, `StatusError` or `StatusUnknown`.

### Optional: standalone server

The SDK also provides `checker.Server`, HTTP scaffolding for running a checker as a remote endpoint instead of (or alongside) an in-process plugin. It exposes the routes `/health` and `/collect`, plus `/definition`, `/evaluate` and `/report` when the provider implements the matching optional interfaces. A provider that implements `CheckerInteractive` (`RenderForm` / `ParseForm`) additionally gets a human-facing form on `/check`, usable outside of happyDomain. See the [SDK README](https://git.happydns.org/checker-sdk-go) for details; the in-process plugin path described above does not require any of this.

---

## Build constraints

Go's `plugin` package is unforgiving. To load successfully, your plugin must be built with:

- the **same Go toolchain version** as happyDomain itself, including the same patch level;
- the **same versions of every shared dependency** (pin them in your `go.mod`, vendoring the exact versions happyDomain ships);
- `CGO_ENABLED=1`;
- the same `GOOS`/`GOARCH` as the host binary.

If any of these do not match, `plugin.Open` fails with a (sometimes cryptic) error like *"plugin was built with a different version of package …"*. The host logs it and skips the file.

Go's `plugin` package only works on **linux**, **darwin** and **freebsd**. On other platforms happyDomain is built without plugin support and the configured plugin directories are ignored, with a warning logged at startup.

---

## Security and deployment

### Directory and file permissions

Loading a `.so` file is arbitrary code execution as the happyDomain process, so the loader enforces strict ownership before it touches any file:

- The plugin directory **must not be a symbolic link** — happyDomain refuses to follow one, to prevent it being redirected to an attacker-controlled path.
- The plugin directory **must not be group- or world-writable**. A directory writable by anyone but the owner is treated as a fatal misconfiguration and aborts loading.
- Any individual `.so` file that is **group- or world-writable is skipped** (logged and ignored), even inside a properly locked-down directory.

In practice: keep the directory owned by the happyDomain user, mode `0755`, and the plugin files mode `0644`.

```bash
sudo install -d -m 0755 -o happydomain /var/lib/happydomain/plugins
sudo install -m 0644 -o happydomain checker-dummy.so /var/lib/happydomain/plugins/
```

### Building the plugin

```bash
CGO_ENABLED=1 go build -buildmode=plugin -o checker-dummy.so ./plugin
```

### Pointing happyDomain at the directory

The directory is configured with the **`--plugins-directory`** flag, which **may be repeated** to scan several directories:

```bash
happydomain --plugins-directory /var/lib/happydomain/plugins
```

The equivalent environment variable is `HAPPYDOMAIN_PLUGINS_DIRECTORY`.

The loader scans each configured directory and attempts to load every `.so` file it finds. An individual plugin that fails to load — wrong build, missing symbols, a panic in its factory — is logged and skipped without aborting startup; one bad `.so` never prevents the others from loading.

### Restart and check the logs

```bash
sudo systemctl restart happydomain
```

On a successful load, happyDomain logs:

```
Plugin com.example.dummy (/var/lib/happydomain/plugins/checker-dummy.so) loaded
```

---

## Licensing

Checker plugins import only `git.happydns.org/checker-sdk-go/checker`, which is licensed under **Apache-2.0**. The SDK is deliberately split out of the AGPL-3.0 happyDomain core as a small, stable public API for third-party checkers.

A plugin built against this SDK is therefore **not** a derivative work of happyDomain, and you may distribute your checker `.so` under any license you choose (MIT, Apache, proprietary, AGPL — whatever fits your needs).

---

## Reference implementation

[`checker-dummy`](https://git.happydns.org/checker-dummy) is the fully working, documented template that this page mirrors. Start from it when writing your own checker.
