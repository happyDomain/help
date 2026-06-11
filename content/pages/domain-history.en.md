---
date: 2025-06-15T11:06:00+02:00
title: "View change history"
weight: 1500
---

Every time you [publish changes]({{% relref "publish-changes" %}}) with happyDomain, they are recorded in a log. This log allows you to easily retrieve the status of your domains as they were previously deployed, and to see when you made each change.

## Opening the history

From your domain's page, open the **History** entry in the menu. happyDomain displays every recorded version of the zone, from the most recent at the top to the oldest at the bottom.

<!-- TODO: screenshot of the history page -->

To keep the list readable, versions are grouped by month. A heading marks the beginning of each month so you can quickly locate a change by its time period.

## Reading a version

Each version is identified by the moment it was last modified, along with the avatar and email address of the author who made the change.

Three dates may be shown for a version:

- **Published on**: when this version was deployed to your DNS provider. A version without this date was saved but never published.
- **Committed on**: when the version was committed (saved as a definitive state of the zone).
- **Modified on**: when the version was last edited.

If a message was attached when the changes were published, it appears below the dates, much like a commit message in version control.

## Viewing the zone at a given time

To inspect the full content of the zone as it was for a given version, click the eye button next to its date. happyDomain opens that version in read-only mode, so you can browse all the records exactly as they were at that moment.

## Comparing two versions

Under each version (except the oldest one), the **View differences** section lets you compare it with the version that immediately precedes it.

<!-- TODO: screenshot of the differences accordion expanded -->

Expand the section to reveal the changes: added records, deleted records, and modified records are highlighted, so you can see at a glance what each publication changed. The most recent comparison is expanded automatically when you open the page.

{{% notice style="info" title="No history yet?" %}}
A domain only builds up a history once you start publishing changes to it. If you have just imported a domain, its history will fill in as you make and publish your first modifications.
{{% /notice %}}
