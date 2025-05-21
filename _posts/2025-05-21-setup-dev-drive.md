---
layout: post
date: 2025-05-21
categories: tips, windows, setup
title: Setting up a Dev Drive on Windows
---

[Dev Drives](https://learn.microsoft.com/en-us/windows/dev-drive/) are a feature on windows that are designed to improve performance while developing. They are optimised to store source code and package caches.

## To Enable

1. Open Windows settings
2. Go to System -> Storage -> Advanced Storage Settings -> Disks & volumes
3. Select `Create Dev Drive`
4. Select `Resize an existing volume`
5. Select drive you want to resize to make the drive
6. Set the size you want to resize the existing drive down to, the remainder will be made available for the dev drive
7. Enter the details for the new Dev drive

The drive will now exist

Check that the drive is trusted `sudo fsutil devdrv query D:`. If it is not a trusted volume it can be enabled with the command `sudo fsutil devdrv trust D:`.

Check that Defender is in performance mode open `Windows Security` -> `Virus & threat protection` -> `Manage settings`. Check that `Dev Drive protection` is On.

## Setup package caches

### Nuget - C#

1. Create new folder on dev drive, `D:\packages\nuget`
2. Create or update the `NUGET_PACKAGES` environment variable to point to folder
3. Update your nuget config to add the folder as a source, located at `$env:APPDATA\NuGet\nuget.config` and add `<add key="Dev cache" value="D:\packages\nuget" />`

### Npm - Node

1. Create new folder on the dev drive, `D:\packages\npm`
2. Create or update the `npm_config_cache` environment variable to point to folder

### Cargo - Rust

1. Create new folder on the dev drive, `D:\packages\cargo`
2. Create or update the `CARGO_HOME` environment variable to point to folder

### PIP - Python

1. Create new folder on the dev drive, `D:\packages\pip`
2. Create or update the `PIP_CACHE_DIR` environment variable to point to folder

### Sources

Finally copy your source code to the new drive. Don't install applications to the dev drive as it is optimised for development not running applications.