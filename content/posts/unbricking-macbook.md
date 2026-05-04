+++
title = "Software update gone wrong - unbricking a Macbook"
date = "2026-05-04T09:17:27+02:00"
#dateFormat = "2006-01-02" # This value can be configured for per-post date formatting
author = "fuzboxz"
authorTwitter = "fuzboxz" #do not include @
cover = ""
tags = ["tech", "personal"]
keywords = ["tech", "apple"]
description = ""
showFullContent = false
readingTime = true
hideComments = false
+++
## Problem

Yesterday I tried updating my Macbook M2 Pro to the latest software version (26.4.1). I usually apply software updates as soon as possible (and things don't go bad — like ever), but I have been struggling with low storage in the last few weeks, so updates haven't been installed. Long story short, I cleaned up ~50 gigs of space, downloaded the update and waited for macOS to do its magic.

To my surprise, when I returned to the machine it was dead: black screen, no light from the keyboard, no boot, no recovery, no startup sound. Panic set in, I realized that the update messed up something that it shouldn't have and turned my notebook into a very stylish paperweight. After a quick search and LLM query I found out that a number of people have experienced similar problems and Apple Geniuses said that the logic board was a goner, requiring replacement and services costing $700. Although in theory I would love to update to the new M5 Max with 128 GB RAM, but waiting for weeks and footing the bill didn't sound fun.


## Solution
When I digged deeper I found that a few people on Reddit managed to revive the machine using Configurator(https://support.apple.com/apple-configurator). This requires an USB-C connection to the DFU port (the top left USB port) of the machine and a backup MacOS device (Macbook, Mac Mini/Studio) to run the app. As I managed Apple MDM previously I was familiar with the app, but this was the first time I used it for something like this. The idea here is that you reflash the low level firmware and hope that it solves the issue.

```
    ┌─────────────────────────────┐         USB-C         ┌─────────────────────────────┐
    │      WORKING MAC            │◄────────────────────►│      BRICKED MAC            │
    │  ┌─────────────────────┐    │    to DFU port       │  ┌─────────────────────┐    │
    │  │  Apple Configurator │    │     (top-left)       │  │       ▄▄▄▄▄▄▄▄▄    │    │
    │  │         ◉           │    │                       │  │       ████ ████    │    │
    │  │  "Reviving device..." │   │                       │  │         💀         │    │
    │  └─────────────────────┘    │                       │  │      (dead)        │    │
    └─────────────────────────────┘                       └─────────────────────────────┘
```

For this to work you have to:
1. disconnect power from the bricked device
2. hook up the bricked device's DFU port to the machine running Configurator
3. while holding Power/TouchID, press Control+Option+Right-Shift for ten seconds and keep holding Power/TouchID for ten more seconds
4. if everything worked correctly you will see the DFU of the bricked machine in the Finder of the secondary machine allowing you to revive (reflash software without erasing data) or recover (full factory reset)
5. you could either reflash the latest software version using Finder or download an Apple signed firmware from ipsw.me and apply using Configurator
6. if you choose Configurator your job is to drag and drop the firmware onto the bricked machine after it has been recognized
Initially I tried reviving through the latest OS version image with Finder, but the problem persisted and the Finder hanged waiting for the bricked device to respond. I figured that it probably a faulty OS update as others had fixed it applying MacOS version 26.3.1, so I decided to download it from ipsw.me and apply it through Configurator. I had to reconnect the device, re-enter DFU mode and downgrade to 26.3.1 using the Revive option.

Luckily this worked, Configurator managed to reflash the older OS and after that bricked device started to boot again. I had to re-unlock the encrypted volume, log in, etc, but luckily all of my apps, settings and files were intact. If a MacOS update bricked your device this might be a way for you to fix it: borrow a second Mac device, connect through USB to the DFU port and try to revert to a previous firmware. 

As for the root cause? Might have been low storage space, low battery, Jupiter in retrograde or a software problem on Apple's side. I don't know, but people should be aware that this could happen and there is possibly a free way to fix it.

## Sources:
* https://support.apple.com/apple-configurator
* https://ipsw.me
* https://www.reddit.com/r/macbookpro/comments/1skw0xd/quick_fix_macos_tahoe_264_2641_killed_my_macbook/
* https://www.reddit.com/r/mac/comments/1snmllz/tahoe_264_bricked_my_m2_macbook_pro_apple_quoted/