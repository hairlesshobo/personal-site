+++
title = 'ccmm (Connection Church Media Manager)'
slug = 'ccmm'
date = "2024-10-22"
lastmod = "2024-11-09"
draft = true
tags = ['utilities', 'projects', 'media', 'video', 'audio', 'production', 'data-management']
keywords = ['home lab', 'go', 'production', 'golang']
description = 'go-import-media, aka gim, is a tool for automatically importing media from removable disks into a predefined folder structure automatically.'
[params]
toc = false
+++

# Connection Church Media Manager (ccmm)

## Introduction

Connection Church Media Manager, aka ccmm, is a tool for managing all aspects of
produced media. The eventual goal is for the tool to import, syncronize and backup
data automatically. The tool is planned to be separated into the following 3 components:

- **Importer:** a tool for automatically importing media from removable disks into a predefined 
  folder structure automatically.
- **Manager:** a daemon that runs on a server to index, organize, receive and synchronize files 
  from multiple sources
- **Client:** an appplication that will be placed on the computer of people in the church who
  need acces to the media. 

## Why?

We generate ~100GB of media data each week at church and currently, I am inserting one
SD card or flash drive at a time and copying to my laptop, then syncing up to the 
server and then finally syncing to the additional storage drives + backblaze. This
is super tedious and wastes a lot of time. 

Additionally, we are growing rapidly and as we grow our volunteer base, it is going to
become impossible to expect others to keep things organized and especially SAFE (think
accidental deletions or files that don't make it back to the archive after being worked with).
While I can't 100% protect from these things, by having a centralized system that manages the
organization and synchronization automatically, I hope to reduce the risk here.

## Importer

### Features


#### Current
  - Auto-mount an external drive (USB or SD card) that was connected to a Linux system via udev
  - Scan the mounted directory to determine if it was produced using a known data source (see supported media sources below)
  - Scan the directory for files that should be imported and gather metadata on them
  - Import any identified files to a predefined folder structure
  - Auto-unmount the external drive, then power off for safe removal
  - Allow importing of media via integrated localsend server. Once a transfer completes, it will follow the usual import process to identify and import media
  - Localsend can be password protected and also supports sender ACLs (not intended for real security, more to prevent accidental ingestion of data)
  - Current support for auto-import on Mac by watching diskutil for inserted disks
  - Sorry, no windows support planned because windows.

#### Planned 
  - Verify enough scratch space prior to accepting transfer in localsend
  - Optionally empty/format a drive after import (will be configurable per-source type)
  - Build and integrate with a microcontroller to provide LED status lights for each card reader/usb port - (this might finally be a reason for me to buy a rasberry pi.. instead of adding an arduino)
  - Add status endpoint to server API for quickly checking status of importer
  - localsend https support
  - logging improvements

### Media sources

#### Supported
  - `behringerX32` - For importing stereo audio recordings created by a Behringer X32
  - `behringerXLIVE` - For importing multi-track audio recordings created by a Behringer X-Live card
  - `blackmagicIOS` - For importing video recordings created by the Blackmagic IOS camera app
  - `canonEOS` - For importing video and photos created by a Canon EOS camera (at least a 60D, not testing on any other yet)
  - `canonXA` - For importing video created by a Canon XA series camcorder, recording in MXF mode (only tested on XA70)
  - `jackRecorder` - For importing multi-track wav files created by the fox-recorder
  - `zoomH6` - For importing multi-track wav files created by the Zoom H6 field recorder

### Planned
  - `zoomH1n` - For importing stereo audio recorded by a Zoom H1n field recorder

### Dependencies

Linux:
- blkid
- lsblk
- findmnt
- udisks2 (for mounting, unmounting, and disk poweroff without sudo access)
- udev
- systemd
- polkit

Mac:
(coming soon)

### Installation

coming soon...

### Usage

coming soon...

## Manager

### Features

coming soon...

### Installation

coming soon...

### Usage

coming soon...

## Client

### Features

coming soon...

### Installation

coming soon...

### Usage

coming soon...

## License

ccmm is licensed under the Apache-2.0 license

Copyright (c) 2024 Steve Cross <flip@foxhollow.cc>

>  Licensed under the Apache License, Version 2.0 (the "License");
>  you may not use this file except in compliance with the License.
>  You may obtain a copy of the License at
>
>       http://www.apache.org/licenses/LICENSE-2.0
>
>  Unless required by applicable law or agreed to in writing, software
>  distributed under the License is distributed on an "AS IS" BASIS,
>  WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
>  See the License for the specific language governing permissions and
>  limitations under the License.

Portions of the code, specifically those providing localsend server functionality, were originally 
written by MeowRain for the [`localsend-go`](https://github.com/meowrain/localsend-go) project. The 
files in question all have the MIT license clearly mentioned in the file header. For the sake of 
simplicity, any additions by Steve Cross to the localsend files are also licensed under the MIT 
license, while the rest of the ccmm project is still licensed under Apache-2.0. For a complete 
copy of the MIT license text, see the `LICENSE-MIT` file in the root of this project. The 
following copyright applies to all localsend-related files only:

Copyright (c) 2024 Steve Cross (additions on or after 2024-10-30)
Copyright (c) 2024 MeowRain

## Links

- [Project on GitHub](https://github.com/hairlesshobo/ccmm/)
- [Project Homepage](https://www.foxhollow.cc/projects/ccmm/)