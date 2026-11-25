---
title: "Introducing FileSpot: File Search for Mac, NAS, and Offline Drives"
date: 2026-11-25
tags: ["filespot", "macos", "nas", "file-search", "products"]
description: "FileSpot has been submitted for Mac App Store review: file-name search for Mac, NAS, and offline drives, with free local search and an optional Pro subscription."
draft: false
---

I have submitted **[FileSpot](https://getfilespot.com/en/)** for review in the Mac App Store and am waiting for Apple's response. It is a macOS app for finding files when they are spread across your Mac, network shares, and external drives.

This is a first look at the product ahead of release. The place to check its availability is [getfilespot.com](https://getfilespot.com/en/).

## The file exists. But on which drive?

Imagine looking for a project document when some folders live on your Mac, others on the office NAS, and older work sits on an external disk. Remembering part of the filename is useful, but you still need to find the right source.

The search built into macOS is often sufficient for local files. Network shares are a different situation, particularly where server-side Spotlight search is unavailable. Disconnected drives introduce another problem: the files cannot be searched live when the source is missing.

FileSpot brings those sources into one search window and shows where each result comes from.

{{< figure src="featured.webp" alt="FileSpot's English search window shows a search for Quote across a local project folder, an office NAS, and an archive drive, with each result's source, size, and date." caption="A real view of FileSpot, using fictional example files." >}}

## Find the name, even when the source is offline

FileSpot searches **file and folder names**. It does not read or index document contents. Filters for type, size, and date help narrow the results, and a keyboard shortcut opens Quick Search from another app.

For supported sources with a local index, FileSpot can search the remembered filenames even when the drive is disconnected. That means you can find where a document was indexed while away from the office or without plugging in an archive disk.

An offline result is a reference to an indexed file, not an offline copy of the document. The original source still needs to be available to open the file. Changes made while the source is disconnected cannot appear until FileSpot observes them again.

For network drives, FileSpot works with shares connected through Finder, typically SMB shares on NAS devices and servers. macOS handles the share's sign-in; FileSpot does not store those passwords.

## See what changed on a share

Search answers where a file is. History helps with a related question: what changed since the previous snapshot?

For each share with a local index, FileSpot takes a daily snapshot while the app is running. Its history shows files that are new, changed, moved, or removed between snapshots.

This is a periodic comparison, not a complete audit of every operation. If several changes happen between snapshots, the history should not be treated as a continuous record of them.

It also does not replace a backup. Knowing where a removed file was last seen can help an investigation, but FileSpot cannot restore its contents. You still need a separate recovery process for that.

## Local search without an account

FileSpot keeps its search data on your Mac. It requires no FileSpot account, uses no cloud search service, and includes no analytics. It only accesses the sources you grant and does not change the files it searches.

That local approach still works with network storage: reading a connected share involves accessing that source, while the optional index stays on the Mac. File contents are neither read nor stored for search.

## Free local search, with Pro for network sources

The free version covers searches on your Mac and connected external drives, filters, Quick Search, and up to three saved searches, with no time limit.

Pro adds network drives and NAS, local indexing and offline search, history, and unlimited saved searches. It will cost **€3.99 per month or €29.99 per year**, with a free trial. These are the listed euro-area prices; the App Store will show local pricing in other countries. Subscriptions can be cancelled through your Apple Account.

The [FileSpot website](https://getfilespot.com/en/) has the full feature comparison and plan details.

## Ahead of the launch

FileSpot requires an **Apple silicon Mac (M1 or later)** and **macOS 26 or later**. It supports English and German.

The app is awaiting review, so there is no confirmed release date yet. I will share the Mac App Store link once it is available. In the meantime, visit **[getfilespot.com](https://getfilespot.com/en/)** for screenshots, product information, and support details.
