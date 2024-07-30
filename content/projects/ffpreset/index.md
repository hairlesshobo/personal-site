+++
title = 'ffpreset'
slug = 'ffpreset'
date = "2024-07-30"
draft = false
tags = ['home lab', 'utilities', 'projects', 'media', 'video', 'audio']
keywords = ['home lab', 'python', 'ffmpeg']
description = 'Python wrapper for ffmpeg that allows you to execute predefined presets and process entire folders'
# summary = 'The overview of my plan to build a bare-metel Kubernetes cluster in the home lab'
[params]
toc = true
+++

## Introduction

I am constantly using ffmpeg for conversion of media files. I was previously using the amazing [FFmpeg batch](https://ffmpeg-batch.sourceforge.io/) tool, but I no longer use Windows as a daily driver and needed something that would allow me to keep track of presets that I frequently use for media captured from different sources, destinations, etc. I tossed together the basic "ffbatch" python script to use as a wrapper that would allow me to define and easily execute presets that I need for various purposes.

Don't expect too much from it - I very much tailored it to my own person needs, but if you have a good idea for it and want to submit a pull request, I am very much open to it!

## Installation

```bash
git clone https://github.com/hairlesshobo/ffpreset.git
cd ffpreset
pip3 install -r requirements.txt
./ffpreset -h
```

## Usage

```
usage: ffpreset [-h] [-a APPEND] [-b] [-c] [--debug] [--dry_run] [-f FILENAME] [-l] [-n] [-o OUTPUT_DIR] [-q] [-s] preset source_file [source_file ...]

Wrapper for ffmpeg that allows to easily run the program using presets

positional arguments:
  preset                The name of the preset to use
  source_file           The full or relative path to the source file to use for encoding

options:
  -h, --help            show this help message and exit
  -a APPEND, --append APPEND
                        Append text to the filename, this overrides whatever is configured in the presets file
  -b, --batch           The specified source is a directory, process all files in it.
  -c, --concat          Concat all provided files
  --debug               Output more logs for debugging
  --dry_run             No action is taken, but the action that WOULD be taken is logged
  -f FILENAME, --filename FILENAME
                        Name to use for the output file. Cannot be used in batch mode
  -l, --list            List available presets
  -n, --normalize       Normalize the audio automatically
  -o OUTPUT_DIR, --output_dir OUTPUT_DIR
                        Specify a directory to write the output file to
  -q, --quiet           Reduce output noise
  -s, --skip_existing   Skip any files that already exist

Run `ffpreset -l` to list available presets
```

## License

This utility is licensed under the MIT license, see the LICENSE file for more details


## Links

[Project on GitHub](https://github.com/hairlesshobo/ffpreset)