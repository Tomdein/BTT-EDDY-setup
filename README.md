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

What I like about this method is that if I set the vibrations just right I can calibrate my printer across the room without even looking at it (I'm sitting with back towards the printer)

## Steps:
0. Find at what temperature you home when you start print (`start_temperature`) and what is the reasonable max temperature the EDDY reaches when printing - you want to calibrate z offset near the temperature you home when printing and do the temperature calibration around this temperature (up to the reasonable max chieavable temperature). Mine setup and startup g-code heats the EDDY to 40°C and more so there is no reason to calibrate it at like 30°C.
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

## Sample configs
### `printer.cfg`
Somewhere near end add these lines as they override some previous values (or just add/merge other .cfg files into printer.cfg)
```
...

[include eddy.cfg]
[include eddy-z-probe.cfg]

...
```

### `eddy.cfg`
```
########################################
# BTT EDDY
########################################
# The MCU section only applies to the Eddy USB. For Eddy Coil you will use the MCU name of the toolboard that you connected the Eddy Coil to.
[mcu eddy]
serial: /dev/serial/by-id/usb-Klipper_rp2040_504450593877391C-if00 # This is the serial address of your eddy probe. This can be found by using the terminal of your klipper instance (typically through SSH) and using the command ```ls /dev/serial/by-id```
restart_method: command

[temperature_sensor btt_eddy_mcu]
sensor_type: temperature_mcu # Sets the type of sensor for Klipper to read
sensor_mcu: eddy # Sets the MCU of the eddy probe tempereature sensor
min_temp: 10 # Sets the minimum tempereature for eddys tempereature sensor to operate
max_temp: 100 # Sets the maximum tempereature for eddys tempereature sensor to operate

[probe_eddy_current btt_eddy]
sensor_type: ldc1612
z_offset: 2.0
#i2c_address:
i2c_mcu: eddy  # This value is good for the Eddy USB but would need to be adjusted for the Eddy Coil according to the MCU you have used.
i2c_bus: i2c0f # This value is good for the Eddy USB but would need to be adjusted for the Eddy Coil according to the I2C bus you have used.
# Measure the offsets below using the method described here: https://www.klipper3d.org/Probe_Calibrate.html#calibrating-probe-x-and-y-offsets
# For a standard Voron stealthburner X carriage mount there should be no need to change the defaults below.
x_offset: 0
# y_offset: 21.42
y_offset: 25.00 # Tom has installed eddy in reverse

# This section is only relevant for Eddy USB. Comment it out for Eddy Coil.
[temperature_probe btt_eddy]
sensor_type: Generic 3950
sensor_pin: eddy:gpio26
horizontal_move_z: 2.0
max_validation_temp: 56

[bed_mesh]
horizontal_move_z: 2.0
speed: 150
# For the mesh dimensions below, the coordinates need to be reachable by the center of the probe. To calculate coordinates that will work, use the formula below:
# mesh x min = position_min_x + greater_of (15mm or x_offset) <--- in this term, only consider the x offset if it is positive, ignore if negative.
# mesh y min = position_min_y + greater_of (15mm or y_offset) <--- in this term, only consider the y offset if it is positive, ignore if negative.
# mesh x max = position_max_x - greater_of (15mm or |x_offset|) <--- in this term, only consider the x offset if it is negative, ignore if positive.
# mesh y max = position_max_y - greater_of (15mm or |y_offset|) <--- in this term, only consider the y offset if it is negative, ignore if positive.
# Example: Consider that you have a 300 x 300 bed with the max x and y positions being 300 and the min being 0. Your probe offsets are -20 for X and 10 for Y
# For mesh x min we ignore the x offset term because it is negative. Therefore mesh x min = 15
# For mesh y min we do not ignore the y offset term because it is positive but it is less than 15 so we use 15. Therefore mesh y min = 15
# For mesh x max we do not ignore the x offset term because it is negative. It is also greater than 15. Therefore mesh x max = 280
# For mesh y max we ignore the y offset term because it is positive but it is less than 15 so we use 15. Therefore mesh y max = 285
# The final result would be mesh_min: 15, 15 mesh_max: 280, 285
# Probe around 15mm from the edge - at 10mm the measurements drop rapidly!
mesh_min: 15, 15 # Can't move more than -12... # modify these according to the above guide. If the probe cannot reach then you will get a klipper error when trying to scan a bed mesh.
mesh_max: 200, 195 # modify these according to the above guide. If the probe cannot reach then you will get a klipper error when trying to scan a bed mesh.
probe_count: 30, 16
mesh_pps: 0, 1
algorithm: bicubic
scan_overshoot: 10  #uncomment this section if you still have room left over on the X axis for some scan overshoot to product smoother movements and more accurate scanning. Uncommenting this should be fine if you are using a standard voron mount.

# Uses probe positions (probe above the screw)
[screws_tilt_adjust]
screw1: 200, 166 # OVERRIDES printer.cfg
screw2: 200, -6 # OVERRIDES printer.cfg
screw3: 15, -6 # OVERRIDES printer.cfg
screw4: 15, 166 # OVERRIDES printer.cfg

# Uncomment this if you are using Eddy as the probe AND the homing endstop
[safe_z_home]
home_xy_position: 100, 100 # Choose an X,Y position that is in the center of your bed. For a 300x300 machine that will be 150, 150. Use the same principle to calculate your bed center.
z_hop: 10
z_hop_speed: 25
speed: 150

###############################Macros and related################################
#This secton contains macros and related config sections. Some should be used, others are optional. Read the comment above each one to find out whether or not to uncomment it for your installation.


# Uncomment this if you are using Eddy as the probe AND the homing endstop AND would like to use the beta z-offset control
#[save_variables]
#filename: ~/printer_data/config/variables.cfg



# Uncomment this if you are using Eddy as the probe AND the homing endstop
[force_move]
enable_force_move: True # Allows a user to move the z axis down if they have no other means of homing Z and need to calibrate the Eddy.



# Uncomment this if you are using Eddy as the probe AND the homing endstop AND would like to use the beta z-offset control
#[delayed_gcode RESTORE_PROBE_OFFSET]
#initial_duration: 1.
#gcode:
#  {% set svv = printer.save_variables.variables %}
#  {% if not printer["gcode_macro SET_GCODE_OFFSET"].restored %}
#    SET_GCODE_VARIABLE MACRO=SET_GCODE_OFFSET VARIABLE=runtime_offset VALUE={ svv.nvm_offset|default(0) }
#    SET_GCODE_VARIABLE MACRO=SET_GCODE_OFFSET VARIABLE=restored VALUE=True
#  {% endif %}



# Uncomment this if you are using Eddy as the probe AND the homing endstop
# Take note of the lines that should only be used if you have a KNOMI installed.
[gcode_macro G28]
rename_existing: G28.1
gcode:
  #SET_GCODE_VARIABLE MACRO=_KNOMI_STATUS VARIABLE=homing VALUE=True # Uncomment this if using a KNOMI and then remove the G28 macro from the KNOMI.cfg
  G28.1 {rawparams}
  {% if not rawparams or (rawparams and 'Z' in rawparams) %}
    PROBE
    SET_Z_FROM_PROBE
  {% endif %}
  #SET_GCODE_VARIABLE MACRO=_KNOMI_STATUS VARIABLE=homing VALUE=False # Uncomment this if using a KNOMI and then remove the G28 macro from the KNOMI.cfg



# Uncomment this if you are using Eddy as the probe AND the homing endstop
[gcode_macro SET_Z_FROM_PROBE]
gcode:
    {% set cf = printer.configfile.settings %}
    SET_GCODE_OFFSET_ORIG Z={printer.probe.last_z_result - cf['probe_eddy_current btt_eddy'].z_offset + printer["gcode_macro SET_GCODE_OFFSET"].runtime_offset}
    G90
    G1 Z{cf.safe_z_home.z_hop}


# Uncomment this if you are using Eddy as the probe AND the homing endstop AND would like to use the beta z-offset control
#[gcode_macro Z_OFFSET_APPLY_PROBE]
#rename_existing: Z_OFFSET_APPLY_PROBE_ORIG
#gcode:
#  SAVE_VARIABLE VARIABLE=nvm_offset VALUE={ printer["gcode_macro SET_GCODE_OFFSET"].runtime_offset }



# Uncomment the lines in this macro if you are using Eddy as the probe AND the homing endstop AND would like to use the beta z-offset control
[gcode_macro SET_GCODE_OFFSET]
rename_existing: SET_GCODE_OFFSET_ORIG
variable_restored: False  # Mark whether the var has been restored from NVM
variable_runtime_offset: 0
gcode:
#  {% if params.Z_ADJUST %}
#    SET_GCODE_VARIABLE MACRO=SET_GCODE_OFFSET VARIABLE=runtime_offset VALUE={ printer["gcode_macro SET_GCODE_OFFSET"].runtime_offset + params.Z_ADJUST|float }
#  {% endif %}
#  {% if params.Z %}
#    {% set paramList = rawparams.split() %}
#    {% for i in range(paramList|length) %}
#      {% if paramList[i]=="Z=0" %}
#        {% set temp=paramList.pop(i) %}
#        {% set temp="Z_ADJUST=" + (-printer["gcode_macro SET_GCODE_OFFSET"].runtime_offset)|string %}
#        {% if paramList.append(temp) %}{% endif %}
#      {% endif %}
#    {% endfor %}
#    {% set rawparams=paramList|join(' ') %}
#    SET_GCODE_VARIABLE MACRO=SET_GCODE_OFFSET VARIABLE=runtime_offset VALUE=0
#  {% endif %}
  SET_GCODE_OFFSET_ORIG { rawparams }



# This macro automates a lot of the frequency mapping process and simplifies the steps significantly.
[gcode_macro PROBE_EDDY_CURRENT_CALIBRATE_AUTO]
gcode:
  BED_MESH_CLEAR
  G28 X Y
  G90 # Abs positioning
  G1 X{ printer.toolhead.axis_maximum.x/2 } Y{ printer.toolhead.axis_maximum.y/2 } F6000
  {% if 'z' not in printer.toolhead.homed_axes %}
    SET_KINEMATIC_POSITION Z={ printer.toolhead.axis_maximum.z-1 } # Allows the user to work it down until it touches.
  {% endif %}
  PROBE_EDDY_CURRENT_CALIBRATE {rawparams}



#This macro is optional but useful if you want to run a rapid scan before each print. Simply uncomment it and add BED_MESH_SCAN to your print start code.
#[gcode_macro BED_MESH_CALIBRATE]
#rename_existing: BTT_BED_MESH_CALIBRATE
#gcode:
#  SET_GCODE_VARIABLE MACRO=_KNOMI_STATUS VARIABLE=probing VALUE=True #Only uncomment this line if using a KNOMI and then remove the BED_MESH_CALIBRATE macro from KNOMI.cfg
#  BTT_BED_MESH_CALIBRATE METHOD=rapid_scan
#  SET_GCODE_VARIABLE MACRO=_KNOMI_STATUS VARIABLE=probing VALUE=False #Only uncomment this line if using a KNOMI and then remove the BED_MESH_CALIBRATE macro from KNOMI.cfg
```

### `eddy-z-probe.cfg`
```
# Klipper config uses the last value so.... If you used position_endstop in main config... comment it
# https://klipper.discourse.group/t/klipper-s-printer-cfg-syntax-and-parsing-rules/23425

# https://global.bttwiki.com/Eddy.html#z-endstop-configuration:
# Under your [stepper_z] in printer.cfg change
# endstop_pin: PA5 to endstop_pin: probe:z_virtual_endstop
# and
# comment out or remove position_endstop: 0.
[stepper_z]
endstop_pin: probe:z_virtual_endstop
```

## Sample startup G-Code
```
G21 ; set units to millimeters
G90 ; use absolute positioning
M82 ; absolute extrusion mode

; Preheat bed
G28 X0 Y0 ; move X/Y to min endstops
G28 Z0    ; move Z to min endstops
G0 Z50    ; Lift up
M104 S180                           ; Set extruder temp to preheat
M140 S{first_layer_bed_temperature[0] + 15 > 115}115{else}{first_layer_bed_temperature[0] + 15}{endif} ; set bed temp to desired temperature + 15°C as my AC bed heats too fast and the bed needs time to soak in.
M190 S[first_layer_bed_temperature] ; wait for bed temp
G4 P90000                           ; Wait 90s for bed to temp soak

; Level, calibrate, etc...
TEMPERATURE_WAIT SENSOR='temperature_probe btt_eddy' MINIMUM=43
BED_MESH_CLEAR
G28 Z0                               ; move Z to min endstops
Z_TILT_ADJUST                        ; Adjust different z1 & z2 heights
G28 Z0                               ; move Z to min endstops
BED_MESH_CALIBRATE METHOD=rapid_scan ADAPTIVE=1 ; Level the bed with BTT EDDY
G28 Z0                               ; move Z to min endstops
G0 Z50                               ; Lift up

; Move nozzle into position and preheat
G0 Z50                          ; Lift up
G0 X0 Y10 Z50 F9000             ; Go to front
M104 S{first_layer_temperature[0]} ; set extruder temp
M109 S{first_layer_temperature[0] - 10} ; wait for extruder temp
; Oozing? Dont wait?; G4 P5000                        ; Wait 5s for extruder to temp soak

; Prime and wipe nozzle
G0 X5 Y10 Z0.4 F9000 ; Drop to bed
G92 E0               ; zero the extruded length
G1 X55 E10 F500      ; Extrude 10mm of filament in a 5cm line
G92 E0               ; zero the extruded length

G0 Z0.15     ; Drop to bed
G1 X80 F4000 ; Quickly wipe away from the filament line
G0 Z0.3      ; Raise and begin printing.
```
