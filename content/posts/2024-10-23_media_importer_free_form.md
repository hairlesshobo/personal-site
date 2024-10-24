+++
title = 'Fully automatic media importing - how?'
date = "2024-10-23"
tags = ['tools','production','media','golang','go','linux']
summary = 'My attempt to create a solution for my automatic media import needs'
+++

## Introduction

Friendly warning: this is going to be very free form and not really worth reading, but overall its an attempt to find an elegant solution to a problem. I don't expect anyone to read it - ever. I am just hoping that writing about it will help me to craft a solution that I don't completely hate and that is reliable enough to bother to build and use. With that warning out of the way, I will begin by describing the problem I am trying to solve.

Each week we produce around 100GB of media data at church. At the moment, that data is produced from a variety of sources..
  * audio recorded on X32 board to a USB flash drive
  * multi-track audio captured via a custom, albeit rough, CLI tool that captures using jack server and currently stored on laptop
  * still photos captured from a Canon DSLR to SD card
  * video captured from a professional Canon camcorder to SD card
  * video captured using the blackmagic camera app on one or more iPhone 14 Pro Max units, stored on device and transferred via localsend

My current process looks like this:
1. Insert each card/drive one at a time and manually copy to proper location in weekly folder structure on laptop
2. Transfer files from iphones using localsend to local folder on laptop
3. Use Forklift app to sync from laptop to "A" archive drive on server (more about this later, if I get to it)
4. Use Forklift app to sync from "A" archive drive on server to "B" archive drive connected to laptop

Sounds simple enough but its a very manual and fairly slow process.

## Goal

So the goal is to create a solution that allows for fully automatic importing of files from the full variety of sources to the correct destination. Eventually I want to expand this to also handle auto archiving to "A" and "B" archives, as well as the backblaze archive. Hardware wise, this part should be pretty straightforward - I will pick up a USB 3 hub and 3-4x SD card readers, plug it all in to a system (likely the HUD unit since its low power, on my desk and powered on during the day when I would need to use it for this). 

To make things a little faster yet still safe, I will pick up a pair of SATA SSD drives, slap them in a mirrored ZFS pool and use them as a quick destination for imported media. I want to use dual SSDs for redundancy and performance. Especially when archiving to multiple destinations, it will be awesome to run it in parallel.

## The main problem

OK so the above goal seems pretty straightforward, but the problem I am having is finding a truly automatic solution for importing. The actual import process will be easy - copy files from the source drive then drop them where they need to go. The issue is identifying what the source is so that it can be placed in the correct destination. I am far too OCD to just blindly import the data into one giant folder. Not. Gonna. Happen.

## Possible solutions

Now we get to the real point of this post - trying to find a solution to automatically and reliably identify the source. 

* **Idea 1:** Use volume UUID \
**Problem:** This UUID changes every time the card is formatted in camera (which is considered best practice).

* **Idea 2:** Use volume label \
**Problem:** If you set this to something custom, in-camera format reverts this back to a default (for example: CANON, NIKON, etc)

* **Idea 3:** Hey! All SD cards have a unique ID that cannot ever change. It is called the CID. Perfect! - or not \
**Problem:** The CID is only readable by an internal card reader that uses the MMC driver. The ioctl calls that must be made in order to read the CID are intercepted by USB card readers and not understood. In short: there is no way to read the CID of a SD card that is connected to a USB card reader. Additionally, even if the CID was readable, this isn't ideal for one reason - we will have to pre-register the CID with the system and then make sure to NEVER swap cards around.

## Current plan

Use a variety of known parameters to identify the source. Current list of parameters that are to be used:
  - directory/file structure - each source uses a predictable directory & file structure. if not, it is disqualified
  - volume label - gets set automatically by cameras on format, if this doesn't match then it is a disqualifier
  - cam serial in XML (for Canon XA70)
  - cam name in exif data (for still cameras)
  
What to do if more than one identical source? For example, we have two Canon EOS cameras of the same model - cam 1 and cam 2, operated by different people, how do we differentiate between the two?
  - Non-issue for Canon XA70 camcorder as this one has serial number recorded in XML sidecar files
  - Non-issue for audio sources - I have no plans to ever have multiple jack recorders or multiple X32 sources of same type (multi-track or stereo recording from board)
  - For still cameras, or video cameras that don't contain unique identifiers, name folders based on identified model. (EOS 60D, XA70, etc). A mostly reliable (hopefully) method for differentiating between multiple sources of the same type would be during scan, identify that the destination already exists and has files that do not match with what is currently on the source. So say we have two 60D cameras, we pop the cards in and it starts processing the first one. It identifies that the source type is "Photo" and the source model is "60D".. it determines that there are photos from 2024-10-20 on the source so it checks `/live/_Services/2024 Q4/2024-10-20/Photo/60D` to see if it exists. If not, it creates it and copies the files to it. If it does exist, it checks to see if any of the files on the source exist in the destination. If existing files are there that DO match the source, it copies any additional files that have no yet been copied. If not a single file matches, it assumes this must be an additional source of the same model and subsequently names the destination `.../Photo/60D_2`.

## Summary

OK I realize this was not anything worth reading, but it helped me come up with a solution to identifying the source so now its just time to build it out in code. I will likely add more to this article when I encounter the next annoying blocker that I need to write through to find a solution.

Oh btw, this is going to be written in Go - most likely.
