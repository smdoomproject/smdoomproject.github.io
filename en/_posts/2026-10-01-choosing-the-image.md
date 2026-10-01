---
title: "Choosing the image: H32 mode, graphics options and colors to polish"
lang: en
---

Since the first post, the project has reached an important milestone: Doom now produces images directly in the Mega Drive's own format. The engine itself builds the tiles, the palettes and the grid that places them, exactly what the console's video chip knows how to display. For now, all of this runs on my Mac, where a small simulator of the video chip displays the result. Not on a Mega Drive yet, not even on an emulator: that will be the next step.

This post tells the story of the choices that led here.

## The real problem: video memory

The Mega Drive's video chip (the VDP) does not draw pixels one by one. It builds the screen out of small 8×8-pixel squares called tiles, stored in its 64 KB of video memory, and each of them can only use 15 colors. In a typical game, these tiles rarely change: a scrolling background reuses the same tiles over and over.

Doom is the exact opposite. As soon as the camera moves, almost the whole image changes. Every new frame therefore needs new tiles, which have to be copied into video memory during the brief moment when the screen is not being drawn. That makes two limits: the total number of tiles the memory can hold, and the number of tiles to transfer between two frames, which caps the frame rate. My first attempt targeted a full-width image, 320 pixels wide. It had to rely on tiles that look alike to fit in memory, which meant making the image poorer… and it still overflowed in more than a third of the frames.

## The Virtua Racing solution

The answer came from a 1994 game: Virtua Racing. Its cartridge contains a special chip, the SVP, which computes the 3D and draws the result directly as tiles, which the console then only has to copy. That is exactly SMDoom's architecture. And Virtua Racing makes a decisive choice: it uses the Mega Drive's narrow display mode, called H32, 256 pixels wide instead of 320.

I made the same choice. SMDoom's view is 256×160 pixels, that is 32×20 = 640 tiles. Every cell of the screen gets its own tile, copied again for every frame. The number of tiles is therefore always the same, whatever happens on screen: video memory can no longer overflow, and image detail no longer costs any memory.

These 640 tiles amount to about 20 KB to copy per frame. By my calculations, the console can receive up to 30 frames per second on 60 Hz systems, and 25 on 50 Hz systems. That is a theoretical ceiling: the real limit will probably be the speed of the coprocessor, which I have not measured yet. But it is above my target of 20 frames per second, compared with 10 to 15 for the Super NES version and 15 to 20 for the 32X version. Fun fact: 256×160 is also the size of the 32X version's view.

A few consequences of this choice:

- H32 pixels are wider than they are tall; without correction, the image would be stretched by about a third. The engine compensates by stretching everything vertically by the same amount;
- the status bar at the bottom of the screen will be redrawn 256 pixels wide and displayed by the console itself, rather than recomputed every frame;
- static screens (title, end of level) keep their original size, 320 pixels: the console switches display mode whenever the screen changes.

## Graphics options to choose from

The Super NES and 32X versions of Doom had to give up some effects, in particular floor and ceiling textures, replaced with flat colors. Rather than settle this once and for all, I split the renderer into settings I can switch on or off: solid or textured floors and ceilings, a dividing line between floor and walls, the number of light levels. More will come: wall textures, shading, color flashes when you get hit or pick up a bonus.

This lets me compare the variants on real game frames. And eventually, I would like to offer some of these settings to the player, so they can choose between a more readable image and a more detailed one.

## Colors, for later

The Mega Drive knows 512 colors, but only shows a few dozen at a time: 4 palettes of 15 colors, and each tile can only use one palette. Doom, on the other hand, has 256 colors and lots of dark gradients. So for every frame, the colors of each palette have to be chosen, and each tile has to be given a palette.

Here is where this work stands.

![Three renderings of the same frame side by side](/assets/images/2026-10-01/compare_1.png)

![Three renderings of the same frame side by side](/assets/images/2026-10-01/compare_2.png)

In each picture, the left image is the reference: every pixel takes the closest of the Mega Drive's 512 colors, with no palette limit at all. That is what we would get if the console could show all its colors at once. The other two are work-in-progress versions, with the real palette constraints: in the middle, all colors count equally; on the right, the algorithm gives priority to enemies and the weapon. You can see it in the muzzle flash, whose white core turns pink in the middle and stays almost white on the right. But you can also see the price: a few floor squares get the wrong color.

The images are enlarged three times, with square pixels: on the console, they will look slightly wider.

There is still a lot to do here. But the way colors are distributed has no effect on the game's speed, and I would rather prove first that the whole thing works. I will come back to colors at the end.

## What's next

Next step: running all of this on an emulated Mega Drive. I am going to modify an emulator so that it simulates a cartridge fitted with a coprocessor, and write the small 68000 program that receives the frames and displays them. It will be the first time SMDoom runs on a Mega Drive, even a virtual one.
