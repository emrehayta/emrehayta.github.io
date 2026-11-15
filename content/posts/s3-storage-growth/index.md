---
title: "Why Your S3 Storage Keeps Growing After You Delete Files"
date: 2026-11-12
tags: ["aws", "s3", "storage", "cloud", "operations"]
description: "Investigate S3 storage growth by separating current objects, noncurrent versions, delete markers, and incomplete multipart uploads before changing lifecycle rules."
draft: false
---

You delete a large set of files from an S3 bucket, but the reported storage does not fall as expected. Before deleting more, check what the report includes and whether the bucket keeps object versions.

The ordinary object listing is only one view of storage. Historical versions and unfinished uploads can explain data that is absent from the application's current file list.

{{< figure src="featured.svg" alt="A cutaway storage bucket shows visible current objects above hidden historical versions and unfinished multipart uploads." >}}

## Establish which measurement is growing

Start with the account, bucket, region, time range, and reporting source. Compare like with like: object counts, stored bytes, and billed cost answer different questions.

Storage metrics and reports may lag recent activity. Request charges, replication, retrievals, and other usage can also change a bill independently of current object bytes. Identify the usage category before assuming a storage cleanup should explain the full cost difference.

For the data investigation, separate these categories:

| Category | What it represents |
| --- | --- |
| Current objects | The versions applications normally see |
| Noncurrent versions | Older object contents retained by versioning |
| Delete markers | Markers that can make a versioned key appear deleted |
| Incomplete multipart uploads | Uploaded parts not yet assembled into a completed object |

Use bucket-level reporting to identify the dominant category, then investigate the affected prefixes. Avoid downloading the entire dataset merely to measure its size.

## Check versioning before interpreting a delete

Using an authorised AWS CLI profile, confirm the account and bucket configuration:

```bash
aws sts get-caller-identity --profile production
aws s3api get-bucket-versioning \
  --bucket YOUR_BUCKET --profile production
```

Replace the profile and bucket with your actual values. These are inspection commands; they do not change the bucket.

In a versioning-enabled bucket, a simple delete generally creates a delete marker rather than removing older object contents. The key disappears from a normal current-object listing, while retained versions continue to consume storage. Removing a specific version is a separate, permanent deletion operation.

Suspending versioning does not clean up existing historical versions. AWS explains delete behaviour in its [versioned object documentation](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DeletingObjectVersions.html).

That distinction matters for frequently overwritten keys. A nightly export uploaded under the same filename can accumulate many historical copies even though the application sees one file.

## Inspect a bounded sample

For a known prefix, list a limited number of version records:

```bash
aws s3api list-object-versions \
  --bucket YOUR_BUCKET \
  --prefix exports/ \
  --max-items 50 \
  --profile production
```

The output may include both versions and delete markers. A bounded sample helps explain behaviour, but it is not a complete storage total. Pagination tokens are evidence that more results exist.

For a large bucket, use an inventory or reporting workflow that covers the versions you need to analyse. A default current-version-only inventory will not answer a question about all historical data. Keep object keys and reports appropriately restricted because filenames can disclose customer or project information.

Identify the writer as well. Retention policy and application behaviour need to agree: a cleanup rule cannot make a continuously growing export stream stop producing data.

## Look for unfinished multipart uploads

Multipart upload allows a client to send an object in pieces. If the upload never completes or is aborted incorrectly by the application, uploaded parts can remain stored without appearing as a completed object.

Inspect uploads for the relevant prefix:

```bash
aws s3api list-multipart-uploads \
  --bucket YOUR_BUCKET \
  --prefix exports/ \
  --max-items 50 \
  --profile production
```

This lists upload records, not the aggregate size of their parts. For a particular upload, `list-parts` can provide more detail. Use S3 reporting for the broad picture rather than treating the number of uploads as a byte count.

First determine whether an upload is genuinely abandoned. An unusually long but legitimate transfer needs a different response from a failed job that never retries or aborts.

AWS documents the cleanup mechanism in its [incomplete multipart upload lifecycle guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpu-abort-incomplete-mpu-lifecycle-config.html).

## Design lifecycle rules around data ownership

Lifecycle configuration is a data deletion policy as well as a storage-management tool. Have the data owner agree to retention, recovery needs, and scope before enabling it.

For example, a team might decide to retain current exports for a business-defined period, retain older versions for a shorter recovery window, and abort incomplete uploads after more time than any supported transfer requires. Those are separate decisions with separate lifecycle actions.

Pay attention to the clock used by each action. Noncurrent-version retention is based on when a version became noncurrent, rather than simply when the object was first uploaded. Current-object expiration in a versioned bucket does not, by itself, guarantee removal of the retained historical data.

Apply policies to a deliberately selected scope and assess their effect on existing objects. Eligible old data can be affected after a rule is enabled. Review Object Lock, retention requirements, and replication arrangements where present. The [lifecycle configuration reference](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intro-lifecycle-rules.html) describes the individual actions.

## Preserve the rules you already have

Before changing lifecycle configuration, inspect and save the existing configuration through your normal infrastructure workflow. An absent configuration is different from a permission error; do not silently treat every failed read as an empty policy.

A lifecycle update can replace the bucket's existing rule configuration. Do not paste a single example rule into a production update without merging and reviewing the other rules. Manage the desired configuration in version control where possible.

Test the policy against an isolated dataset and confirm the expected results. Lifecycle processing is asynchronous, so an immediate listing is not a complete test of the eventual effect. AWS's [configuration guide](https://docs.aws.amazon.com/AmazonS3/latest/userguide/how-to-set-lifecycle-configuration-intro.html) covers setup and verification.

## Close the loop with the application

After the reporting window catches up, compare the storage categories again. Document which writer, versioning behaviour, or unfinished transfer caused the growth and whether the retention change matches the agreed recovery window.

Then fix the source where appropriate: stop redundant exports, abort failed multipart transfers, or choose intentional naming and retention behaviour.

Version history can help recover an accidental overwrite, but a usable recovery process still needs access, scope, and testing. The same questions from a [restore drill]({{< ref "backup-restore-testing" >}}) apply here: which point can you recover, and how would you prove it?
