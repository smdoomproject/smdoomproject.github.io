---
title: "Let's talk frame rate"
lang: en
---

The project is moving along nicely, and it's time to pause for a short technical update. One crucial point in this Doom port is the frame rate. On PC, Doom runs at 35 frames per second. On the Super Nintendo, it gets between 10 and 15, and on the 32X between 15 and 20. But how many could I actually get?

## How it works

The architecture is now settled: the RP2350, the cartridge's coprocessor, renders a frame and writes it to a memory chip on the cartridge (the SRAM). It then hands over to the Mega Drive, which copies that data into its video memory (the VRAM). There are two SRAMs: while the Mega Drive copies a frame from one, the RP2350 prepares the next one in the other.

So there are three steps: rendering the frame, writing it to SRAM, and copying it to VRAM. Writing to SRAM should be quick, and can only be measured on a physical prototype. The other two steps, however, can be measured right now, and need to be. Only once those numbers are known can I move on to the emulator, so that it reproduces the cartridge's behavior as faithfully as possible.

That's why I changed the order announced in the previous post: before modifying an emulator, I need to know the timings it will have to match.

## Resolution

The Mega Drive has two horizontal display modes: H40, with a 320-pixel-wide image, and H32, with a 256-pixel-wide image. Still screens will be sent in H40, since they don't need a high data rate. That covers the title screen, the screens between levels and at the end of the game, as well as the menus: opening a menu displays it in H40 on a black background, and the game returns to the previous screen when it closes. The demos that play behind the menu in the original Doom are therefore disabled. Levels, on the other hand, will be sent in H32 to limit the amount of data to generate and transfer, with the status bar (the HUD) stored in the ROM and drawn by the console itself.

## On the Mega Drive side

I've made good progress on the ROM, the program that runs on the console's processor. The goal is to get conditions as close as possible to the final version. It can therefore display still screens in H40, and levels in H32 with the status bar added. For now, I'm testing it in the BlastEm emulator, with generated test images.

On every mode change, a fade to black replaces Doom's "melt" effect, where the image drips down the screen. During the fade, Doom pauses: the transition costs the player no time, however long it takes.

And the first numbers are in.

The Mega Drive refreshes the screen 60 times per second in NTSC (Japan, North America, where it's sold as the Genesis), versus 50 times in PAL (Europe). These numbers can't be changed. The same image will therefore stay on screen until the next one is ready. Using the test images, I found that transferring one level frame takes 3 screen refreshes in NTSC, and only 2 in PAL. The difference comes from the fact that the console can only copy data to VRAM quickly between two images, while the screen isn't being drawn: in PAL, the screen has more lines for the same visible image, so that pause is much longer.

This means the frame rate could reach 20 frames per second in NTSC (60/3) and 25 in PAL (50/2). In the previous post, I estimated the ceiling at 30 frames per second in NTSC: the measurement brings it down to 20, which is still my minimum target. In PAL, the 25 frames per second are confirmed.

These numbers only apply to the display: the game itself keeps running at its normal speed, it just shows fewer frames.

But they only hold if the RP2350 renders its frames faster than that, which we won't know until further tests.

## Latency

Another important point: latency. How much time passes between the moment the RP2350 starts rendering a frame and the moment the Mega Drive displays it? Assuming the RP2350 keeps up with the Mega Drive, 20 frames per second in NTSC means one frame every 0.05 s. The RP2350 renders a frame while the Mega Drive transfers the previous one: a frame therefore spends 0.05 s being rendered, then 0.05 s being transferred. The latency is thus 0.1 s between the start of rendering and the display. In PAL, it's 0.08 s.

## What's next

Next step: measure how long the RP2350 takes to render a frame. The tests will run on a Raspberry Pi Pico 2, starting with getting Doom to run on it.

For the final version of the ROM, what's left is handling the color flashes in the status bar and redrawing the faces.
