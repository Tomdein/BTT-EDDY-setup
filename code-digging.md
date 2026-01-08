# I started code spelunking again...
So I looked at the [source code](https://github.com/Klipper3d/klipper/blob/e605fd18560fbb5a7413ca12b72325ad4e18de16/klippy/extras/temperature_probe.py#L477-L721) again and this is how the calibration works

## Paper test
Instead of the "paper test" I do a "sound test":
- Heat the nozzle to woking temperature (230-250°C) and pull the filament so it does not ooze. Clean the nozzle
- Lower the nozzle quickly by eye (or paper test) and then by 0.1mm, then 0.05mm, then 0.025mm (can even do 0.01mm) until you ear the nozzle transmit the vibrations into the bed (the bed will resonate)
- Now, if you move the nozzle 0.01mm lower it should resonate just a bit more. You are done
- Profit

## Z offset calibration:
Using the the "paper or sound test" you calibrate the Z offset.

I like to heat the nozzle to working temperature (230-250°C) and touch the nozzle to bed using "sound test" - there are no assumptions about therma expansion as in [klipper wiki](https://www.klipper3d.org/Bed_Level.html#the-paper-test:~:text=Always%20perform%20the,the%20calibration%20process.):
```
Always perform the paper test when both nozzle and bed are at room temperature!

When the nozzle is heated, its position (relative to the bed) changes due to thermal expansion. This thermal expansion is typically around a 100 microns, which is about the same thickness as a typical piece of printer paper. The exact amount of thermal expansion isn't crucial, just as the exact thickness of the paper isn't crucial. Start with the assumption that the two are equal (see below for a method of determining the difference between the two distances).

It may seem odd to calibrate the distance at room temperature when the goal is to have a consistent distance when heated. However, if one calibrates when the nozzle is heated, it tends to impart small amounts of molten plastic on to the paper, which changes the amount of friction felt. That makes it harder to get a good calibration. Calibrating while the bed/nozzle is hot also greatly increases the risk of burning oneself. The amount of thermal expansion is stable, so it is easily accounted for later in the calibration process.
```

## Temperature calibration:
I can recommend setting [`max_validation_temp`](https://www.klipper3d.org/Config_Reference.html#temperature_probe) as [the default is 60°C](https://github.com/Klipper3d/klipper/blob/e605fd18560fbb5a7413ca12b72325ad4e18de16/klippy/extras/temperature_probe.py#L493) and I can barely achieve 57°C after quite a long time (maybe 60°C mid print, but you can cool down the probe in seconds or a minute back to like 55°C). So do not waste the fitting range of parabolas if you are not gonna use it.

This is from the klippy.log (nicer than saved in printer.cfg):
```
temperature_probe btt_eddy: loaded temperature drift calibration. Min Temp: 52.72, Min Freq: 3177684.129537
y(x) = 41.453572x^2 - 4462.573284x + 3338643.898176
y(x) = 40.226843x^2 - 4370.559797x + 3327207.705152
y(x) = 23.667902x^2 - 2545.410015x + 3269084.935903
y(x) = 22.763332x^2 - 2440.136236x + 3259832.486524
y(x) = 18.548743x^2 - 1970.447002x + 3241777.790841
y(x) = 18.174786x^2 - 1928.159609x + 3236644.440851
y(x) = 18.512105x^2 - 1968.344761x + 3234657.337628
y(x) = 17.773036x^2 - 1886.599914x + 3229803.424028
y(x) = 19.769154x^2 - 2101.837087x + 3233549.941562
temperature_probe btt_eddy: registered drift compensation with probe [probe_eddy_current btt_eddy]
Extruder max_extrude_ratio=0.281963
```
It creates [9 fitting parabolas](https://github.com/Klipper3d/klipper/blob/e605fd18560fbb5a7413ca12b72325ad4e18de16/klippy/extras/temperature_probe.py#L483) at 9 different Z offsets (let's call it a Z discretization).

For every Z distance (of the 9) a sample is taken with "paper or sound test" at given temperature. At the end for every Z (of the 9) the correspoonding probed points are fitted using parabola.

Therefore I recommend that the temperature calibration does have samples around the temperature where the Z offset was calibrated (look in the printer.cfg at the bottom). Still there is a fit function so it might be off a little?

The temperature calibration should not invalidate the Z offset calibration? Maybe if you have done the temperature calibration over a long time and do not wanna repeat the process, just calibrate the Z offset? Or just use the [beta Z offset feature...](https://www.klipper3d.org/Bed_Level.html#the-paper-test:~:text=Always%20perform%20the,the%20calibration%20process.) 

### How do I understand the code:
[This is the important code](https://github.com/Klipper3d/klipper/blob/e605fd18560fbb5a7413ca12b72325ad4e18de16/klippy/extras/temperature_probe.py#L680-L714) that will explain it all, mainly [this part](https://github.com/Klipper3d/klipper/blob/e605fd18560fbb5a7413ca12b72325ad4e18de16/klippy/extras/temperature_probe.py#L701-L714).

These are [the parabolas](#temperature_calibration) from temperature calibration:

Let's start from the Z offset calibration:
You tell the firmware where the Z=0mm is (via paper/sound test) and then the probe does a lot of measurements at different Z distances:

Then you do the temperature calibration and you take the [9 Z samples](https://github.com/Klipper3d/klipper/blob/e605fd18560fbb5a7413ca12b72325ad4e18de16/klippy/extras/temperature_probe.py#L483) at different temperatures (the hues of yellow-red) that are then fitted to parabolas:

When you then do a Home Z, the probe does look at the value from the probe (light green point) and takes the current probe's temperature. Then it goes and finds the parabola above and under the point and takes the "distance" from both of them (30% and 70%).
Now the algorithm goes and looks up what value should the probe have at the Z offset calibration temperature (many many blue points) and it gets the dark green point. All that is left is to compare the calculated dark green point to known measured/calibrated Z offsets (blue points).

Ta daaah, you have the resulting Z offset with temperature calibration.

# Recommendations:
- Calibrate Z offset near the working temperatures ([blue hue](#how_do_i_understand_the_code)). Home Z near these temperatures.
- Do not run long operations (z tilt? long mesh generation?) while the probe is heating up (like 30°C and the stable temperature is 50°C)
- Basically for ideal results stay near the Z offset calibration temperature ([blue hue](#how_do_i_understand_the_code))
- Make sure you probe the around the Z offset calibration temperature ([blue hue](#how_do_i_understand_the_code)) while doing the temperature calibration
- Lower the [max_validation_temp](#temperature_calibration) if you will not reach 60°C during Home Z
- When doing the probe temperature calibration (TEMPERATURE_PROBE_CALIBRATE PROBE=btt_eddy TARGET=<fill your max temp> STEP=1), set the STEP=1 or STEP=2 and do as the calibrations when you want based on the nozzle temperature. That is mainly because when the probe is at like 35°C the temperature rises sooo fast it is at like 40°C while I wanted the temperature at 35°C. So get ready before and do just the tiny adjustment when the temperature is right
- You should be able redo the Z offset calibration alone, but remember! If you want to get the nozzle closer to the bed, move it a bit further from the bed when doing Z offset calibration (if you do it by the sound method = nozzle touches the bed, then move a bit up (like a paper width) or so) ??TODO check. It makes sense in picture??:
- 
