---
date: 2026-04-05T12:00:00+02:00
title: Monitoring & Checks
author: nemunaire
weight: 35
description: "Set up automated health checks to monitor your domains and services"
---

happyDomain includes a built-in monitoring system that lets you run automated health checks on your domains and services. Checkers periodically collect data (such as ping response times, domain expiry dates, or DNS audit results), evaluate it against thresholds you define, and report a clear status: OK, Warning, Critical, or Error.

## Browsing available checkers

You can see all available checkers by navigating to the **Checkers** page from the main menu.

<!-- TODO: screenshot of the checkers list page -->

Each checker is labelled with the scope it applies to:

- **Domain-level**: checks that concern the domain itself, independent of DNS records (e.g. domain expiry via WHOIS).
- **Zone-level**: checks that need the full zone content (e.g. DNSSEC validation).
- **Service-level**: checks that target a specific service on a subdomain (e.g. ping, HTTP check).

Use the search bar to filter checkers by name. Click on a checker to see its description and configuration options.


## Configuring a checker for your domain

To set up a checker on one of your domains:

1. Go to your domain's page and open the **Checks** tab.
2. You will see a table of checkers available for this domain. Click **Configure** next to the checker you want to set up.

<!-- TODO: screenshot of the domain checks list -->

On the configuration page, you will find several sections:

### Checker options

Options are grouped by category:

- **Admin Options** -- system-wide settings controlled by the administrator (read-only for regular users).
- **Configuration** -- user-level preferences such as warning and critical thresholds. These are the main settings you will adjust.
- **Domain-specific Settings** -- values automatically filled in based on the domain being checked (e.g. the domain name itself).
- **Checker Parameters** -- additional runtime parameters.

Fill in or adjust the options to match your needs, then click **Save**.

<!-- TODO: screenshot of the checker configuration page with options -->

### Rules

Each checker comes with one or more **rules** that evaluate different aspects of the collected data. For example, a ping checker might have separate rules for latency and packet loss.

You can enable or disable individual rules using the toggle switches. Each rule may also have its own specific options that you can configure.

<!-- TODO: screenshot of the rules section with toggles -->


## Scheduling automatic checks

Once a checker is configured, you can schedule it to run automatically at a regular interval.

In the **Schedule** section of the checker configuration page:

1. Toggle **Run automatically** to enable scheduling.
2. Set the **Check interval** (in hours) -- this determines how often the check runs.
3. The **Next scheduled run** time is displayed so you know when the next execution will happen.
4. Click **Save** to apply the schedule.

<!-- TODO: screenshot of the schedule card -->

{{% notice style="info" title="Interval limits" icon="clock" %}}
Each checker defines minimum and maximum intervals to prevent overloading external services. The interface will respect these bounds when you configure the schedule.
{{% /notice %}}

When scheduling is disabled, the checker can still be run manually at any time.


## Running a check manually

You can trigger a check at any time without waiting for the schedule:

1. Navigate to the checker's **Executions** page (from the domain checks list, click **View Results**).
2. Click **Run Check Now**.
3. A dialog opens where you can:
   - Select which rules to run for this execution.
   - Override options temporarily (these overrides are not saved).
4. Click **Run Check** to start the execution.

<!-- TODO: screenshot of the Run Check modal -->

A confirmation message will appear with the execution ID. The result will show up in the executions list once the check completes.


## Viewing check results

All past executions are listed on the checker's **Executions** page, accessible from your domain's Checks tab by clicking **View Results**.

<!-- TODO: screenshot of the executions list -->

The executions table shows:

| Column | Description |
|--------|-------------|
| **Executed at** | When the check ran |
| **Status** | The result: OK, Warning, Critical, Error, or Unknown |
| **Message** | A summary of the findings |
| **Duration** | How long the check took to complete |
| **Type** | Whether it was a scheduled or manual run |

Click **View** on any execution to see its detailed results.


## Understanding execution details

The execution detail page presents the results in the most appropriate format depending on what the checker provides.

### Metrics

When a checker produces metrics (e.g. response time, packet loss percentage), the detail page shows:

- A **line chart** plotting the metric values over time across recent executions.
- A **data table** listing each metric with its name, value, and unit.

<!-- TODO: screenshot of the metrics chart and table -->

### HTML reports

Some checkers generate rich HTML reports (e.g. detailed DNS audit results). These are rendered directly in the page.

### Raw data

You can always switch to the **Raw JSON** view to inspect the full observation data collected during the execution. Use the view toggle at the top of the report section to switch between Metrics, HTML Report, and Raw JSON.

### Rule evaluations

Each rule that ran during the execution reports its own status and message. You can see the per-rule breakdown to understand which specific aspect triggered a warning or critical status.

<!-- TODO: screenshot of rule evaluation results -->


## Service-level checks

Checkers that apply at the service level are configured from the service's own page rather than the domain-level Checks tab.

1. Navigate to your domain, then to the specific subdomain and service.
2. Open the **Checks** tab for that service.
3. The workflow is the same as for domain-level checkers: configure options, set up scheduling, run checks, and view results.

Service-level checkers automatically receive information about the service they are attached to (such as IP addresses from a Server service), so they require less manual configuration.


## Managing check results

From the executions list, you can:

- **Delete a single result** by clicking the delete action on a specific execution.
- **Delete all results** for a checker using the bulk delete option (a confirmation dialog will appear).

This can be useful to clean up old data or reset after configuration changes.


## Status reference

Checkers report one of the following statuses, in order of severity:

| Status | Meaning |
|--------|---------|
| **OK** | Everything is within acceptable parameters |
| **Info** | Informational finding, no action needed |
| **Warning** | A threshold is approaching; attention recommended |
| **Critical** | A threshold has been exceeded; action required |
| **Error** | The check itself failed (collection error, bad configuration) |
| **Unknown** | The check could not determine a result |
