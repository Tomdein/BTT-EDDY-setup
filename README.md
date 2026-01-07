# BTT-EDDY-setup
How to properly set up BTT EDDY probe

## Setup:
Follow [BTT EDDY wiki](https://global.bttwiki.com/Eddy.html) ([wayback backup](https://web.archive.org/web/20251225225843/https://global.bttwiki.com/Eddy.html)) to set up the EDDY (I used the probe + z-endstop config (no beta z-offset))

## Problem:
First layer is almost great... but it detaches. After few tries I got a bit that did not stick to the nozzle and I measured the thickness around 0.4-0.6mm based on where and how hard I squished callipers. Let's call it a 0.5-0.6mm first layer. I print at 0.35 first layer... Something is offset by few tenths of mm.

Changing z_offset in [probe_eddy_current btt_eddy] does not work at all - seemingly does nothing (probabl used internaly for calculations and then it gets cancelled out? I only know that after homing, the probe moves and sets z=z_offset)

Home the Z axis and measure using feeler gauge - got around 0.12-0.15 (pretty nice - almost paper thickness). Now if you think about it a little... 0.15 + 0.35 = 0.5... Too similar measured print.

When you dig in the klipper & BTT wikis you find that:
### Klipper:
```
Always perform the paper test when both nozzle and bed are at room temperature!

When the nozzle is heated, its position (relative to the bed) changes due to thermal expansion. This thermal expansion is typically around a 100 microns, which is about the same thickness as a typical piece of printer paper. The exact amount of thermal expansion isn't crucial, just as the exact thickness of the paper isn't crucial. Start with the assumption that the two are equal (see below for a method of determining the difference between the two distances).

It may seem odd to calibrate the distance at room temperature when the goal is to have a consistent distance when heated. However, if one calibrates when the nozzle is heated, it tends to impart small amounts of molten plastic on to the paper, which changes the amount of friction felt. That makes it harder to get a good calibration. Calibrating while the bed/nozzle is hot also greatly increases the risk of burning oneself. The amount of thermal expansion is stable, so it is easily accounted for later in the calibration process.
```

### BTT:
[Temperature Compensation Calibration (Eddy USB ONLY)](https://global.bttwiki.com/Eddy.html#temperature-compensation-calibration-eddy-usb-only)
```
...
4.This will cause the UI to display the z axis adjustment box. Use the paper method mentioned here to pinch a sheet of paper between the nozzle and the bed and then accept the value.
5.After accepting the value, turn on your heat bed to the maximum value and your nozzle to 220℃.
...
```

That's contradictory...

# FIX
- Follow BTT wiki when setting up the EDDY probe
- When doing paper test (with heated nozzle? every time?) do it as usual - find where paper just starts lightly dragging and then move the nozzle a bit lower so it drags more.
- Move down by 0.1-0.15mm so that the probe's Z=0 is when the probe touches the bed and not when it is touching the paper
- Profit

# TODOs:
- Command for mesh generation is: BED_MESH_CALIBRATE METHOD=rapid_scan (don't use the METHOD=scan SCAN_MODE=rapid)
- Do not probe closer than 15mm from edge! Dropoff starts for me between 10-15mm (or check it yourself)
- Explain z_offset - Basically unused, but this is the distance at which the probe can scan the lowest

# Your TODOs:
- Check if you need to move down by 0.1-0.15mm at different nozzle temperatures (thermal expansion) and decide on nozzle temperature when doing calibrations (cant cold calibrate while doing EDDY's temperature compensation calibration)
- Find the right value in from the 0.1-0.15mm (yours might be more or less)
