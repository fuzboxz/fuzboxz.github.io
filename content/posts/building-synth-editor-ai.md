+++
title = "Six hours, sixty knobs - building a synth editor with pi coding agent"
date = "2026-07-01T21:00:00+02:00"
author = "fuzboxz"
authorTwitter = "fuzboxz"
cover = ""
tags = ["tech", "ai", "personal"]
keywords = ["tech", "ai", "synth", "midi", "web-midi"]
description = "How I built a full web-based preset editor for a hardware synth in an afternoon using an AI coding agent."
showFullContent = false
readingTime = true
hideComments = false
+++
## Preface

I have a [Doboz Audio DVNA](https://doboz.audio/products/dvna) — an awesome chord/texture synthesizer with around sixty controllable parameters mapped to four knobs, four buttons and a tiny OLED screen. It is wonderful to play, but somewhat hard to program using the constrained physical hardware interface. What I wanted was a proper editor: a web page where I can see every parameter at once, drag sliders, randomize parameters, save and load patches the easy way.

 ```
      ┌──────────────────────────────────────────────────────────────────────┐
      │  USB-C         MIDI IN          LINE OUT         HEADPHONES          │
      │  (5V power)    (TRS A / B)      (stereo)         (stereo)            │
      ├──────────────────────────────────────────────────────────────────────┤
      │  ╭───────────╮       ( ○ )    ( ○ )    ( ○ )    ( ○ )      ◄ pots    │
      │  │ ▖▗ ▝▘ ▗▖ │         P1       P2       P3       P4                  │
      │  │  4-OSC   │       [ A ]    [ B ]    [ C ]    [ D ]      ◄ backlit  │
      │  ╰───────────╯         ◉        ◉        ◉        ◉                  │
      │   ◄─ OLED               chord & texture synth                        │
      ├──────────────────────────────────────────────────────────────────────┤
      │   ║║    ║║    ║║    ║║    ║║    ║║                                   │
      │   ║║    ║║    ║║    ║║    ║║    ║║                                   │
      │   ║║    ║║    ║║    ║║    ║║    ║║              ◄ row 2 (top)        │
      │    F#    G    G#    A    A#     B                                    │
      │   ║║    ║║    ║║    ║║    ║║    ║║                                   │
      │   ║║    ║║    ║║    ║║    ║║    ║║                                   │
      │   ║║    ║║    ║║    ║║    ║║    ║║              ◄ row 1 (bottom)     │
      │    C    C#     D    D#     E     F                                   │
      └──────────────────────────────────────────────────────────────────────┘
 ```

Previously I might have waited until someone else released an editor, but given how much dev work I did in the recent months I decided to just vibe it. It's a good self-contained project with a well-defined end state, something that I can do while I'm taking a break from my *other things*.

## Exploration

In my projects I **always** start with exploration and a proof-of-concept. Before I spend an ungodly amount of time wiring up all of your parameters and finetuning UI quirks I first have to validate if my core idea is actually feasible. For this I try to explore the technical limitations of the approach I'm taking, what can I do, what can't I do, what workarounds exist, etc. "Coding" agents are perfect for this: if you point them at the product documentation, a firmware, tell them what you want and ask nicely they will take care of hours of deep research. Once this is done I come up with a limited scope PoC to validate that my assumptions before I start doing the actual work.

Here the process was similar. Luckily MIDI CC is so simple, that there was no question about whether that works or not. However this still didn't mean that I didn't face a couple of interesting limitations during the exploration phase: 
1. DVNA has TRS MIDI-IN, but no OUT/THRU
2. DVNA USB-C is power only
3. DVNA has presets in Disk Mode, however this format isn't documented

This meant that bi-directional SysEX is out of question and Disk Mode presets take more work than I wanted to initially put into this project. I *could have* reverse engineered the file format of course, however that would introduce problems long-term if the format ever changes. I didn't want to reverse engineer the firmware after every change or accidentally force Mate from [Doboz](https://doboz.audio/) into a position where he has to maintain a legacy format because users of my tool rely on it.

## Architecture

The whole idea behind the project was to create a small, offline-friendly web editor using WebMIDI. Web technology means that it can run offline on any/every platform both locally or hosted and I also wanted to deploy to Github Pages, so that people don't need to install it. LLMs are better at figuring out what the latest hot JS technology is than I am, so I described what I was looking for and the key restrictions like automated tests, secure-by-design, Github Pages deploy using Github Actions, local first or WebMIDI.

```
   ┌───────────────────────────────┐   Web MIDI   ┌───────────────────────────────┐
   │         BROWSER (Chrome)       │◄────────────►│            DVNA                │
   │  ┌─────────────────────────┐   │   CC only    │  ┌─────────────────────────┐   │
   │  │  56 sliders, tabs, 🎲    │   │   ────────►  │  │   listens, never replies │   │
   │  │  save/load .json         │   │  (no sysex)  │  │       (MIDI-IN only)     │   │
   │  └─────────────────────────┘   │              │  └─────────────────────────┘   │
   └───────────────────────────────┘              └───────────────────────────────┘
```

## Implementation

The implementation broke down roughly like this:

1. 0:00–1:00 — PoC: create app scaffolding (Svelte + Vite + TypeScript), wire up Web MIDI, find channel and test if slider changes are propegated to DVNA.
2. 1:00–2:00 — MVP: all 56 parameters mapped, clamped, and streaming correctly, including the 14-bit ones (MSB+LSB).
3. 2:00–3:00 — UI improvements: pulled the firmware binary apart, disassembled it, and dug out the real units behind the knobs (exponential time tables, soft-clip curve). Could not recover Hz/dB, but got enough to scale the sliders sensibly.
4. 3:00–4:00 — New features: randomize, MIDI panic, file-based preset save/load, and a mock mode so I could test without the device plugged in.
5. 4:00–5:00 — Making it production ready: match DVNA aesthetics, security review, accessibility review.
6. 5:00–6:15 — CI/deploy: a least-privilege GitHub Actions pipeline, CodeQL scanning, and deploying it to Pages.

## So, how fast is fast?

In roughly six hours I shipped what would have previously been a couple of weekends. I didn't have to wait for a closed-source vendor locked implementation, I was able to come up with a FOSS web-based alternative, while I was chilling on the couch. Also given that I take quality seriously I was able to deliver things that are missing from usual quick & dirty hobby projects: accessibility, security and a proper UI.

Do I care about not writing a single line of code? No, it would have taken more time, the outcome wouldn't have been better and it's just a tiny bit more complex than a basic CRUD application. 

## Sources
* https://fuzboxz.github.io/dvna-editor/
* https://github.com/fuzboxz/dvna-editor
* https://developer.mozilla.org/en-US/docs/Web/API/Web_MIDI_API
* https://svelte.dev/
* https://vite.dev/
