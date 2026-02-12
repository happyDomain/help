---
title: "Writing a happyDomain Plugin"
description: "Technical guide for developing test plugins for happyDomain"
---

happyDomain supports external **test plugins** — shared libraries (`.so` files) that add domain or service health checks to a running instance. Plugins are loaded at startup without recompiling the server; the operator simply drops a `.so` file into a configured directory.

## How it works

A plugin receives a set of options assembled from several configuration scopes, runs a check (HTTP call, DNS query, …), and returns a result with a status level and an optional detailed report. Results are stored and displayed in the happyDomain UI alongside the domain or service they concern.

When happyDomain starts it scans every directory listed in the `plugins-directories` configuration option. For each file it finds, it:

1. Opens the shared library.
2. Looks up the exported symbol `NewTestPlugin`.
3. Calls `NewTestPlugin()` to obtain a plugin value.
4. Registers the plugin under each name returned by `PluginEnvName()`.

If the file is not a valid Go plugin, if `NewTestPlugin` is missing, or if it returns an error, a warning is logged and the file is skipped. The server always starts regardless of individual plugin load failures.

---

## The `TestPlugin` interface

Every plugin must implement four methods:

```go
type TestPlugin interface {
    PluginEnvName() []string
    Version()          PluginVersionInfo
    AvailableOptions() PluginOptionsDocumentation
    RunTest(PluginOptions, map[string]string) (*PluginResult, error)
}
```

---

## Project structure

A plugin is a standalone Go module compiled with `-buildmode=plugin`. It must be in `package main` and export exactly one symbol:

```go
func NewTestPlugin() (happydns.TestPlugin, error)
```

Recommended layout:

```
myplugin/
├── go.mod
├── Makefile
└── plugin.go       # (or split across multiple .go files)
```

### go.mod

```
module git.happydns.org/happyDomain/plugins/myplugin

go 1.25

require git.happydns.org/happyDomain v0.0.0
replace git.happydns.org/happyDomain => ../../
```

The `replace` directive points to your local happyDomain checkout, ensuring the plugin is compiled against the exact same types as the server.

{{< notice style="warning" >}}
A Go plugin and the host process share the same runtime. They **must** be compiled with the same Go toolchain version and the same versions of every shared dependency. Any mismatch produces a hard error at load time.
{{< /notice >}}

---

## Entry point

```go
package main

import "git.happydns.org/happyDomain/model"

func NewTestPlugin() (happydns.TestPlugin, error) {
    return &MyPlugin{}, nil
}
```

The constructor is a good place to perform one-time initialisation (open config files, create an HTTP client, …). Return an error if the plugin cannot function.

---

## Naming — `PluginEnvName()`

Returns one or more short, lowercase identifiers. These names are used to look up the plugin via the API and to key its stored configuration.

```go
func (p *MyPlugin) PluginEnvName() []string {
    return []string{"myplugin"}
}
```

Choose names that are unlikely to collide (e.g. `"zonemaster"`, `"matrixim"`) and keep them **stable across versions** because they are persisted alongside user configuration. If two loaded plugins claim the same name, the second one is skipped and a conflict is logged.

---

## Version and availability — `Version()`

Describes the plugin and controls where it appears in the UI:

```go
func (p *MyPlugin) Version() happydns.PluginVersionInfo {
    return happydns.PluginVersionInfo{
        Name:    "My Plugin",
        Version: "1.0",
        AvailableOn: happydns.PluginAvailability{
            ApplyToDomain:    true,
            ApplyToService:   false,
            LimitToProviders: nil,  // nil or empty = all providers
            LimitToServices:  []string{"abstract.MatrixIM"},
        },
    }
}
```

| Field | Type | Description |
|---|---|---|
| `ApplyToDomain` | `bool` | Plugin can be run against a whole domain |
| `ApplyToService` | `bool` | Plugin can be run against a specific DNS service |
| `LimitToProviders` | `[]string` | Restrict to certain DNS provider identifiers (empty = no restriction) |
| `LimitToServices` | `[]string` | Restrict to certain service type identifiers, e.g. `"abstract.MatrixIM"` (empty = no restriction) |

Both `ApplyToDomain` and `ApplyToService` may be `true` simultaneously.

---

## Options — `AvailableOptions()`

Options are key/value pairs (`map[string]any`) that configure each test run. They are declared grouped by **scope**, i.e. who sets them and how long they persist:

```go
func (p *MyPlugin) AvailableOptions() happydns.PluginOptionsDocumentation {
    return happydns.PluginOptionsDocumentation{
        RunOpts:     []happydns.PluginOptionDocumentation{ /* … */ },
        ServiceOpts: []happydns.PluginOptionDocumentation{ /* … */ },
        DomainOpts:  []happydns.PluginOptionDocumentation{ /* … */ },
        UserOpts:    []happydns.PluginOptionDocumentation{ /* … */ },
        AdminOpts:   []happydns.PluginOptionDocumentation{ /* … */ },
    }
}
```

### Option scopes

| Scope | Who sets it | Storage key | Typical use |
|---|---|---|---|
| `RunOpts` | User, at test time | _(transient)_ | Per-invocation parameters |
| `ServiceOpts` | User | plugin + user + domain + service | Service-level configuration |
| `DomainOpts` | User | plugin + user + domain | Domain-level configuration |
| `UserOpts` | User | plugin + user | Personal preferences (e.g. language) |
| `AdminOpts` | Administrator | plugin | Instance-wide settings, shared credentials |

Before `RunTest` is called, happyDomain merges all scoped values from least specific (admin) to most specific (run-time). More-specific values silently override less-specific ones. `RunTest` always receives a single flat map and does not need to know which scope each value came from.

### Option fields

Each option is a `PluginOptionDocumentation` (an alias for `Field`):

| Field | Type | Description |
|---|---|---|
| `Id` | `string` | **Required.** Key used in the `PluginOptions` map inside `RunTest` |
| `Type` | `string` | Input type: `"string"`, `"select"` |
| `Label` | `string` | Human-readable label shown in the UI |
| `Placeholder` | `string` | Placeholder text for the input field |
| `Default` | `any` | Default value pre-filled in the form |
| `Choices` | `[]string` | Options for `"select"` inputs |
| `Required` | `bool` | Whether the field must be filled before running |
| `Secret` | `bool` | Marks the field as sensitive (e.g. an API key) |
| `Hide` | `bool` | Hides the field from the user entirely |
| `Textarea` | `bool` | Renders a multiline text area |
| `Description` | `string` | Help text displayed below the field |
| `AutoFill` | `string` | Populate the field automatically from context (see below) |

### Auto-fill

When `AutoFill` is set, happyDomain populates the field from the test context; the user is not prompted:

| Constant | String value | Populated with |
|---|---|---|
| `happydns.AutoFillDomainName` | `"domain_name"` | FQDN of the domain under test, e.g. `"example.com."` |
| `happydns.AutoFillSubdomain` | `"subdomain"` | Subdomain relative to the zone, e.g. `"www"` — service-scoped tests only |
| `happydns.AutoFillServiceType` | `"service_type"` | Service type identifier, e.g. `"abstract.MatrixIM"` — service-scoped tests only |

```go
{
    Id:       "domainName",
    Type:     "string",
    Label:    "Domain name",
    AutoFill: happydns.AutoFillDomainName,
    Required: true,
}
```

---

## Running the check — `RunTest()`

`RunTest` receives the merged option map and a metadata map (reserved for future use), performs the check, and returns a `PluginResult`.

Always assert option values to a concrete type before use — the map holds `any`:

```go
func (p *MyPlugin) RunTest(opts happydns.PluginOptions, _ map[string]string) (*happydns.PluginResult, error) {
    domain, ok := opts["domainName"].(string)
    if !ok || domain == "" {
        return nil, fmt.Errorf("domainName option is required")
    }

    // … perform the check …

    return &happydns.PluginResult{
        Status:     happydns.PluginResultStatusOK,
        StatusLine: "All good",
        Report:     myStructuredReport,
    }, nil
}
```

Return a **non-nil error** only for unexpected failures (network errors, invalid configuration). For expected check failures — the monitored service is down, DNS records are wrong — return a `PluginResult` with an appropriate status and a human-readable `StatusLine`.

### Result fields

| Field | Type | Description |
|---|---|---|
| `Status` | `PluginResultStatus` | Overall result level (see below) |
| `StatusLine` | `string` | Short summary displayed in the UI |
| `Report` | `any` | Any JSON-serialisable value stored as structured diagnostic data |

### Status levels (worst → best)

| Constant | Meaning |
|---|---|
| `PluginResultStatusKO` | Check failed |
| `PluginResultStatusWarn` | Check passed with warnings |
| `PluginResultStatusInfo` | Informational, no action required |
| `PluginResultStatusOK` | Check fully passed |

---

## Building

```bash
go build -buildmode=plugin -o happydomain-plugin-test-myplugin.so \
    git.happydns.org/happyDomain/plugins/myplugin
```

Minimal `Makefile`:

```makefile
PLUGIN_NAME=myplugin
TARGET=../happydomain-plugin-test-$(PLUGIN_NAME).so

all: $(TARGET)

$(TARGET): *.go
	go build -buildmode=plugin -o $@ git.happydns.org/happyDomain/plugins/$(PLUGIN_NAME)
```

The prefix `happydomain-plugin-test-` is a convention; happyDomain loads every file in the plugin directories regardless of its name.

---

## Deployment

### 1. Copy the `.so` file

```bash
cp happydomain-plugin-test-myplugin.so /usr/lib/happydomain/plugins/
```

### 2. Point happyDomain at the directory

`happydomain.conf`:

```
plugins-directories=/usr/lib/happydomain/plugins
```

Environment variable:

```bash
HAPPYDOMAIN_PLUGINS_DIRECTORIES=/usr/lib/happydomain/plugins
```

Multiple directories may be listed as a comma-separated value.

### 3. Check the logs

On a successful load:

```
Plugin My Plugin loaded (version 1.0)
```

On a name conflict or load error a warning is logged with the filename and reason.

---

## Reference implementations

Two plugins are bundled in this directory:

- **`matrix/`** — queries the Matrix federation tester API. Demonstrates `ApplyToService` with `LimitToServices` and `AdminOpts` for the backend URL.
- **`zonemaster/`** — drives the Zonemaster JSON-RPC API, polls for completion, and maps results to severity levels. Demonstrates `AutoFillDomainName`, `UserOpts` for language selection, and multi-level status mapping.
