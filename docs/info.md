<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.

You can also include images in this folder and reference them in the markdown. Each image must be less than
512 kb in size, and the combined size of all images must be less than 1 MB.
-->


# SiliDize

## Introduction

This project attempts to use most of the available IO on the Tiny Tapeout board. The circuit implements a pseudo-RNG using DFFs in a Linear Feedback Shift Register (LFSR). The clock provides the trigger, and the RESET button resets the internal DFF outputs (Qs).

The output of the RNG is limited to 1–6 to emulate a six-sided die, with the final stage of the circuit being a simple binary-to-7-segment display converter.

## How it works

The system can be broken down into three parts:

1. LFSR chain
2. Value limiter
3. 7-segment converter

The **LFSR Chain** consists of gated DFFs, allowing an initial seed to be entered for the random function. This seed is limited to being non-zero (the first DFF is always held at 1).

The RESET button acts as a toggle key, using a DFF configured as a toggle flip-flop. The RNG is considered active when the `.` on the display is ON. If the `.` is off, the system is in RESET mode or SEED SELECTION mode, wherein the DIP switches set the seed value.

The LFSR is "tapped" using an XOR circuit set to a feedback polynomial, mathematically chosen to optimize for **maximum-length sequence**. This provides a good statistical spread; however, the value limiter somewhat undermines this — though in practice it should still perform homogeneously over the probability distribution. This feedback is fed back into the chain.

The **Value Limiter** was computed using a simple K-Map simplification.

Finally, the **Binary to 7-Segment Display Converter** displays the "dice" value to the user.

> **Note:** IN0 on the DIP switch is unused. This is because accidentally setting the seed value to 0 would cause the RNG to stall, which is why the first bit of the RNG seed is always set to 1.

## How to test

The Tiny Tapeout TT07 demo board features:

1. 8 DIP switches
2. RESET button
3. CLK STEP button
4. CLK waveform generator (via onboard RP2350B)
5. 7-segment display

Once the system has finished booting up, the dice can be used as follows:

1. **Getting a Random Number**
   - Use the SPDT switch to choose between STEP or CLK as the trigger source for the RNG.
   - Once the trigger completes (a single press of the STEP button, or via the clock generator), the number is shown on the display.

2. **Seed Selection**
   - Enter `SEED SELECTION` mode by pressing the RESET button; the `.` on the display will turn OFF.
   - Pick a seed by setting or unsetting SW2–SW8 on the DIP switch array.
   - Press the RESET button again to load the seed into the RNG, then use it normally (see *Getting a Random Number*).

## Final Thoughts

Improvements to the RNG mainly concern the value limiter section. This could be enhanced by conditionally forcing another trigger on the LFSR until a number within the target range is met. However, this is difficult to implement in Wokwi, but could be a good exercise if written in HDL.
