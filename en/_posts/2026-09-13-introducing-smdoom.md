---
title: "Introducing SMDoom"
lang: en
---

I'm trying to get Doom running on a stock Sega Mega Drive, using a coprocessor inside the cartridge — in the spirit of what Sega did with the SVP chip in Virtua Racing. I picked the RP2350 as the coprocessor, the chip found in the Raspberry Pi Pico 2, among other boards.

## Why this is hard

The Mega Drive's 68000 CPU and VDP graphics chip were never meant to run a real-time 3D rendering engine. Every serious attempt to port Doom-like rendering to 16-bit hardware runs into the same wall: not enough CPU power, not enough video memory, and a display pipeline built for scrolling 2D tiles, not a freely moving 3D view.

The approach here is to let the coprocessor do the actual 3D computation (the Doom engine's logic, projection, visibility), then convert its output into something the Mega Drive's video chip can display natively: tiles, palettes, and scroll values it already knows how to send to the screen. The 68000's job becomes receiving that data and forwarding it to the VDP, rather than computing the 3D scene itself.

## Where things stand

This is a personal, spare-time project, still in the exploration phase. Several fundamental questions aren't settled yet: how to fit a 3D view into the Mega Drive's tile and color budget without exceeding its video memory limits, how fast the coprocessor can realistically stream data across the cartridge port, and how close the result can get to something genuinely playable rather than a technical curiosity running at a handful of frames per second.

I don't know yet if this will work. That uncertainty is part of why I wanted a public devlog rather than a big announcement: I'd rather document the real process — dead ends included — than let it seem like the outcome was obvious from the start.

## Goals

I'll consider the project successful if I manage to get Doom running on a real Mega Drive, with graphics quality and a frame rate good enough for enjoyable play. So I'm aiming for at least 20 frames per second.

## What to expect here

Short, occasional posts as real milestones are reached: a rendering approach that finally holds together, a frame rate measurement, a problem that turned out to be harder (or easier) than expected. No fixed schedule.

The full source code will be made public later, once the project is far enough along. A link will be added here at that point.
