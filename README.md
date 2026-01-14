# BTT-EDDY-setup
How to properly set up BTT EDDY probe. I assume you have read the BTT EDDY wiki and you set up the probe and you are struggling to get nice first layer.

I haven't tried the Beta Z-offset functionality so this guide might be useless if you use the beta feature. From what I saw in the added g-codes it might solve the improper probing.

While calibrating the EDDY and following this 'guide' take caution as always!!!

Before you start calibrating the probe, read on how I probe the nozzle instead of `paper method` below (and if you are corious the difference between `PROBE_EDDY_CURRENT_CALIBRATE_AUTO` and `TEMPERATURE_PROBE_CALIBRATE` in [code-digging.md](code-digging.md)):

## How to probe:
I think this is the crucial step and the reason many people have problem with probing. It is because the temperature calibration must be done with heated nozzle so the EDDY heats up and the `paper method` assumes the cold probing as the nozzle thermaly expands about the thicknes of a paper:

```
Always perform the paper test when both nozzle and bed are at room temperature!

When the nozzle is heated, its position (relative to the bed) changes due to thermal expansion. This thermal expansion is typically around a 100 microns, which is about the same thickness as a typical piece of printer paper. The exact amount of thermal expansion isn't crucial, just as the exact thickness of the paper isn't crucial. Start with the assumption that the two are equal (see below for a method of determining the difference between the two distances).

It may seem odd to calibrate the distance at room temperature when the goal is to have a consistent distance when heated. However, if one calibrates when the nozzle is heated, it tends to impart small amounts of molten plastic on to the paper, which changes the amount of friction felt. That makes it harder to get a good calibration. Calibrating while the bed/nozzle is hot also greatly increases the risk of burning oneself. The amount of thermal expansion is stable, so it is easily accounted for later in the calibration process.
```

So either lower the nozzle ~0.1mm after you are satisfied or use other methods like vibration probing:

## Vibration probing:
Achieve true Z=0 when probe just touches the bed, I attached a (bit broken) fan to the extruder so it vibrates and when the nozzle touches the bed the bed starts to resonate. Therefore I can immediately know when the nozzle is at Z=0. It takes a few tries to know wheather to use 0.05mm or 0.025mm,.. steps or wheater to add 0.01mm step after you are satisfied as you might be using a textured PEI sheet...

TODO: Video

## Steps:
0. Find at what temperature you home when you start print (`start_temperature`) and what is the reasonable max temperature the EDDY reaches when printing - you want to calibrate z offset near the temperature you home when printing and do the temperature calibration around this temperature (up to the reasonable max chieavable temperature)
1. Read and setup the probe as in [BTT EDDY wiki](https://global.bttwiki.com/Eddy.html)([wayback backup](https://web.archive.org/web/20251225225843/https://global.bttwiki.com/Eddy.html)) to set up the EDDY (I used the probe + z-endstop config (no beta z-offset)). You ca stop after step with `LDC_CALIBRATE_DRIVE_CURRENT CHIP=btt_eddy + SAVE_CONFIG`

## Z offset steps:
1. Make sure the nozzle is clear and you pulled the filament out so there is no oozing, your printer is aligned mechanically (as it should be for calibrating and printing - use Z-Tilt adjust, use calipers to measure Z1 and Z2 steppers, etc...) and maybe set higher timeout `SET_IDLE_TIMEOUT TIMEOUT=36000` (my X-Gantry drops a little when motors turn off)
2. Home all axes
3. Preheat all to the temperature you print at normally - maybe let bed soak a minute, turn the nozzle heater with disconnected fan (it heats the EDDY so fast I do not have time to probe Z=0 at all when the heatsink fan is on)
4. When nearing the `start_temperature` do the `PROBE_EDDY_CURRENT_CALIBRATE_AUTO CHIP=btt_eddy` command with desired probing method

## Temperature calibration steps:
It is similar to Z offset calibration, just try to be faster and have a sample or two before EDDY reaches `start_temperature`
1. Make sure the nozzle is clear and you pulled the filament out so there is no oozing,  your printer is aligned mechanically (as it should be for calibrating and printing - use Z-Tilt adjust, use calipers to measure Z1 and Z2 steppers, etc...) and maybe set higher timeout `SET_IDLE_TIMEOUT TIMEOUT=36000` (my X-Gantry drops a little when motors turn off)
2. Home all axes
3. Preheat all to the temperature you print at normally - maybe let bed soak a minute, turn the nozzle heater with disconnected fan (it heats the EDDY so fast I do not have time to probe Z=0 at all when the heatsink fan is on)
4. Run `TEMPERATURE_PROBE_CALIBRATE PROBE=btt_eddy TARGET=<your max> STEP=2` as soon as posible (even when the bed/nozzle is heating) as the probe will move to position and you have time to lower the nozzle closer to the bed so you do not waste time spamming -1mm, -1mm, -1mm, -1mm, -0.1mm, -0.1mm, -0.1mm, -0.1mm, ...
5. Get ready to accept the the first sample a few degrees before your first desired temperature as the probe samples a few seconds and at the beginning the EDDY's temperature rises rapidly.
6. Use the desired probing method and accept samples. When the temperature is at 30-50°C degrees be as fast as possible as the probe heats quickly. That's why I use `STEP=2` even though I maight sample realisticly at steps of 4°C.

## Z offset vs temperature calibration:
TLDR:
- Z offset calibration sets Z heights to real sensors values (like Z=0.050000 -> 3218598.476, Z=1.770000 -> 3191836.152, ...)
- Temperature calibration creates a lookup table that when the sensor reads 3191836.152 @55°C it translates it to for example 3192252.280 @43°C where the Z offset calibration took place. Thats why you want samples starting before Z offset calibrated temperature and ending at whatever max achievable.

Therefore I conclude that Z offset and temperature calibrations are INDEPENDENT - If your Z offset is off a little, all you need to do is to do ONLY Z offset calibration (I think to lower the nozzle during print, you need to probe a little bit further - I think its because the probe's values drop with distance).

BUT BE CAREFUL: you must probe in the range of temperature calibrated values! Even when printing (use TEMPERATURE_WAIT SENSOR='temperature_probe btt_eddy' MINIMUM=xyz) as when outside the range, the probe [defaults to only Z offset calibration](https://github.com/Klipper3d/klipper/blob/48f0b3cad6d4593746384bf49a39913dcb8cc796/klippy/extras/temperature_probe.py#L692) which you moved up a little and therefore there might be a gap... So maybe just redo both...

To read more on how it work read [code digging](code-digging.md).

# How I found this method:
This is like 3rd rewrite of me figuring things out.

First I started with [README.old.md](README.old.md). You may read it but it is the least refined one and quite a mess really.

If you want to know more how the temperature calibration works go read the [code digging](code-digging.md).
