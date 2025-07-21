# FLSUN S1 PRO Klipper Configuration

This repository contains Klipper configuration files and custom macros for the FLSUN S1 PRO delta 3D printer with Direct Drive Extruder (DDE).

## ⚠️ IMPORTANT: Macro Name Changes

Due to Klipper parser limitations, all macros starting with "S1_PRO" have been renamed to "SPRO":
- Use `SPRO_PRINT_END` instead of ~~S1_PRO_PRINT_END~~
- Use `SPRO_PRINT_START` instead of ~~S1_PRO_PRINT_START~~
- Use `SPRO_FULL_CALIBRATION` instead of ~~S1_PRO_FULL_CALIBRATION~~
- And so on for all other macros

**Why?** Klipper's parser interprets "S1" at the start of macro names as a G-code command, causing "Unknown command: S1" errors.

## ⚡ Recent Configuration Improvements (2025-07-20)

Major configuration improvements have been implemented to resolve critical issues and optimize performance:

### 🔧 Critical Fixes Applied:
1. **Pressure Advance Optimization** - Updated pressure advance for direct drive extruder (was incorrectly configured for bowden)
2. **Dual-Zone Bed Heating** - Fully implemented independent control of inner and outer heating zones
3. **Power Loss Recovery** - Fixed shutdown sequence that prevented proper recovery
4. **Sensor Detection** - Optimized motion sensor sensitivity for direct drive system
5. **Safety Improvements** - Resolved relay_off macro that was incorrectly disabling the extruder
6. **Screen Interface Fix** - Fixed SMART_LOAD blocking screen animation by adding proper completion signal

### 🚀 New Features:
- **M141/M191 Commands** - Control outer bed zone like standard bed commands
- **BED_TEMPS Macro** - Set both bed zones with a single command
- **Enhanced Z-Offset Range** - Increased flexibility for different build surfaces
- **Full Fan Power** - Removed artificial 55% limitation on part cooling fan

See the [Configuration Changes](#configuration-changes-2025-07-20) section below for detailed technical information.

## Important Printer-Specific Information

### Filament Path and Sensors
The FLSUN S1 PRO uses a dual-sensor system with a Direct Drive Extruder (DDE):
- **Filament Sensor Location**: Mounted near the filament spool holder at the top of the printer
- **Motion Sensor**: Detects filament movement to identify clogs or grinding
- **Switch Sensor**: Changes state when filament passes through
- **Filament Guide Tube**: Approximately 700-800mm long PTFE tube from sensor to direct drive extruder (NOT a bowden extruder - this is just a guide tube)
- **Direct Drive Extruder**: The extruder motor is mounted directly on the print head, providing precise filament control

### Build Specifications
- **Build Plate**: Textured PEI plate with maximum temperature of 120°C
- **Build Volume**: Φ320mm × 430mm (cylinder with 383mm maximum height)
- **Heating System**: Smart zone heating system for precise temperature control and improved print quality

### Critical Operating Procedures

#### To Unload Filament:
1. Heat the hotend to appropriate temperature
2. Open the filament guide tube connector located 10cm above the extruder
3. Run `SMART_UNLOAD` macro
4. Pull the filament out of the extruder when prompted
5. Close the filament guide tube connector

#### To Change Filament:
1. Open the filament guide tube connector 10cm above the extruder
2. Run `SMART_UNLOAD` macro and pull filament out
3. Cut the extracted filament at a 45-degree angle
4. Remove the 10cm piece from the extruder and discard
5. Manually retract remaining filament by rolling the spool
6. Insert new filament into the sensor at the top
7. Push filament through the guide tube past the connector into the extruder
8. Run `SMART_LOAD` macro
9. Close the filament guide tube connector

### Why Sensor Disabling is Critical
During filament loading/unloading operations, the sensors must be disabled because:
- Filament movement through the sensors during these operations would trigger false runout alarms
- The motion sensor would interpret the loading/unloading movement as normal printing, potentially masking real issues
- Without disabling, the printer would pause mid-operation thinking filament has run out

## Macro Overview

The configuration files contain comprehensive macros organized into the following categories:

### 1. **Configuration and Settings**
- `_SPRO_SETTINGS` - Central configuration for all macro parameters

### 2. **Filament Management**
- `SMART_LOAD` - Intelligent filament loading with sensor management
- `SMART_UNLOAD` - Safe filament unloading procedure
- `LOAD_[MATERIAL]` - Material-specific loading (PLA, PETG, ABS, ASA, TPU, PC)
- `UNLOAD_[MATERIAL]` - Material-specific unloading
- `LOAD` / `UNLOAD` - Quick access aliases for SMART_LOAD/UNLOAD
- `CLEAN_FILAMENT` - Purge 160mm of filament through nozzle (usage: `CLEAN_FILAMENT S=240`)
- `M600` - Filament change pause

### 3. **Sensor Control**
- `ENABLE_SENSORS` - Enable both filament sensors
- `DISABLE_SENSORS` - Disable both sensors for maintenance
- `CHECK_SENSORS` / `SENSOR_STATUS` - Diagnostic tool for sensor status

### 4. **Temperature Management**
- `SPRO_MATERIAL_SETTINGS` - Apply material-specific temperatures and Z-offset
- `SPRO_HEAT_ZONES` - Smart dual-zone heating for materials
- `PID_HOTEND` - Hotend PID calibration
- `PID_BED` - Bed PID calibration (both zones)

### 5. **Bed Leveling & Calibration**
- `SPRO_BED_LEVEL` - Enhanced bed leveling with temperature preservation
- `BED_LEVEL_1` / `bed_level_1` - Delta calibration
- `BED_LEVEL_2` / `bed_level_2` - Bed mesh calibration
- `SPRO_MESH_MANAGER` - Temperature-based mesh management
- `SPRO_FULL_CALIBRATION` - Complete calibration sequence
- `ZUP` / `ZDOWN` - Z-offset fine adjustment (±0.025mm)
- `CALIBRATE_MOTOR` - Motor calibration routine
- `MEASURING_RESONANCES` - Input shaper calibration

### 6. **Print Management**
- `SPRO_PRINT_START` - Comprehensive print initialization
- `SPRO_PRINT_END` - Safe print completion procedure
- `SPRO_PRIME_LINE` - Draw prime line for nozzle priming
- `START_PRINT` - Power loss recovery aware print start
- `END_PRINT` - Power loss recovery aware print end
- `PAUSE` / `RESUME` - Print pause/resume with position save
- `CANCEL_PRINT` - Cancel print with proper shutdown

### 7. **Dual-Zone Bed Control**
- `M141` - Set outer bed zone temperature (usage: `M141 S70`)
- `M191` - Wait for outer bed zone to reach temperature
- `BED_TEMPS` - Set both zones at once (usage: `BED_TEMPS INNER=60 OUTER=55`)

### 8. **LED & Output Control**
- `screen_led_on` / `screen_led_off` - Control screen RGB LEDs
- `box_led_on` / `box_led_off` - Control enclosure LED
- `laser_on` / `laser_off` - Control laser output
- `relay_on` / `relay_off` - Control relay output
- `drying_box_1` / `drying_box_off` - Control filament dryer

### 9. **Maintenance & Utilities**
- `SPRO_MAINTENANCE_MODE` - Enter safe maintenance mode
- `TMC` - Dump TMC driver information
- `save_time` - Track total print time
- `WAIT_FOR` - Wait with progress updates
- `TIMELAPSE` - Timelapse photography marker

### 10. **Power Loss Recovery**
- `SAVE_POWER_LOSS_PARAMS` - Save print state
- `RESUME_INTERRUPTED` - Resume after power loss
- `PRE_RESUME_INTERRUPTED` - Prepare for resume
- `START_PRINT_RESUME` - Complete resume process
- `SHUTDOWN` - Emergency shutdown

## Detailed Macro Descriptions

### _SPRO_SETTINGS
**Purpose**: Centralized configuration storage for all macro parameters  
**Variables**:
- `extruder_temp_*`: Temperature settings for each material
- `bed_temp_*`: Bed temperature settings for each material  
- `z_offset_*`: Material-specific Z-offset adjustments
- Load/unload distances and speeds for multi-stage operations
- Prime line configuration for delta printer

### SMART_LOAD
**Purpose**: Intelligently loads filament with automatic sensor management  
**Parameters**: 
- `TEMP` (optional): Override temperature (default: 220°C)
**Process**:
1. Heats extruder to specified temperature
2. Automatically disables sensors to prevent false triggers
3. Multi-stage loading sequence optimized for direct drive:
   - Initial feed (10mm @ 5mm/s)
   - Prime extruder (15mm @ 10mm/s)
   - Final extrusion (5mm @ 2.5mm/s)
4. Disables extruder motor (signals completion to screen)
5. Maintains hotend temperature for subsequent operations
6. Prompts user to re-enable sensors after completion

### SMART_UNLOAD
**Purpose**: Safely unloads filament with enhanced tip-forming to prevent jams  
**Parameters**:
- `TEMP` (optional): Override temperature (default: 235°C)
**Process**:
1. Heats extruder to specified temperature
2. Disables sensors automatically
3. Enhanced 4-stage unloading with tip forming:
   - Tip forming: Small push (2mm) and retract to prevent stringing
   - Cooling moves: Multiple retracts/pushes to shape tip properly
   - Clear hotend: Controlled retraction (10mm @ 30mm/s)
   - Final retract: Fast movement away from gears (10mm @ 50mm/s)
4. Prompts for manual sensor re-enabling
**Key Improvement**: Prevents "mushroom" tip formation that causes gear jams

### LOAD_[MATERIAL] / UNLOAD_[MATERIAL]
**Purpose**: Material-specific wrappers for smart load/unload with preset temperatures  
**Available Materials**: PLA, PETG, ABS, ASA, TPU, PC  
**Features**:
- Automatically uses optimal temperature for each material
- Ensures consistent loading/unloading conditions
- Reduces user error in temperature selection

### ENABLE_SENSORS / DISABLE_SENSORS
**Purpose**: Unified control of both filament sensors  
**Affected Sensors**:
- `filament_sensor`: Primary runout detection
- `filament_motion_sensor`: Movement/clog detection
**Usage Notes**:
- Always disable before filament changes
- Re-enable before starting prints
- Both sensors controlled simultaneously for safety

### CHECK_SENSORS
**Purpose**: Diagnostic tool for troubleshooting sensor issues  
**Information Provided**:
- Sensor enable/disable state
- Filament detection status
- Motion sensor status
- Helpful for debugging false triggers or missed detections

### CLEAN_FILAMENT
**Purpose**: Purge filament through the nozzle for cleaning or color changes  
**Parameters**:
- `S` (required): Temperature in Celsius (e.g., `CLEAN_FILAMENT S=240`)
**Process**:
1. Heats extruder to specified temperature
2. Extrudes 100mm at moderate speed
3. Extrudes additional 60mm at slower speed (total 160mm)
4. Turns off hotend after completion
**Usage**: Perfect for purging old colors or materials after filament change

### SPRO_MATERIAL_SETTINGS
**Purpose**: Applies comprehensive material-specific settings  
**Parameters**:
- `MATERIAL`: One of PLA, PETG, ABS, ASA, TPU, PC
**Settings Applied**:
- Extruder temperature
- Bed temperature (both inner and outer zones)
- Z-offset adjustment for material characteristics
**Usage**: Call before printing or when switching materials

### SPRO_BED_LEVEL
**Purpose**: Enhanced bed leveling routine preserving temperatures  
**Parameters**:
- `BED_TEMP` (optional): Override bed temperature
- `EXTRUDER_TEMP` (optional): Override extruder temperature  
**Features**:
- Saves current temperatures before leveling
- Performs delta calibration and bed mesh
- Restores original temperatures
- Shows bed mesh visualization

### SPRO_PRINT_START
**Purpose**: Comprehensive print preparation sequence  
**Parameters**:
- `BED_TEMP`: Target bed temperature
- `EXTRUDER_TEMP`: Target extruder temperature
- `MATERIAL` (optional): Material type for automatic settings
**Sequence**:
1. Initial heating to 80% of target
2. Home and QGL/bed mesh
3. Final heating to target
4. Sensor enablement check
5. Prime line routine (delta-optimized)
6. Ready for print

### SPRO_PRINT_END
**Purpose**: Safe print completion and cooldown  
**Features**:
- Coordinated retraction
- Safe Z-lift within delta limits
- Park position at X0 Y-140
- Controlled cooldown of all heaters
- Motor idle timeout

### SPRO_FULL_CALIBRATION
**Purpose**: Complete automated calibration sequence for the S1 PRO - performs ALL calibrations  
**Process** (in order):
1. **Motor Calibration** - Calibrates all three delta motors
2. **Hotend PID Tuning** - Tunes PID values at 240°C
3. **Bed PID Tuning** - Tunes both bed zones at 60°C
4. **Input Shaper Calibration** - Measures resonances and configures input shaping
5. **Delta Calibration** - Calibrates delta geometry
6. **Bed Mesh Profiles** - Creates meshes for 60°C, 70°C, 80°C, 90°C, 100°C
   - Each temperature includes 2-minute heat soak
**Duration**: Approximately 45-60 minutes
**When to use**:
- Initial printer setup
- After mechanical changes
- Monthly maintenance
- If print quality degrades
**Note**: Requires Klipper restart after completion to apply all calibrations

### SPRO_MAINTENANCE_MODE
**Purpose**: Enter safe maintenance mode for manual operations  
**Actions**:
- Disables all filament sensors
- Turns off all heaters
- Sets conservative motion limits (100mm/s velocity, 1000mm/s² acceleration)
- Disables extruder stepper
**Usage**: Run before performing manual maintenance like nozzle changes or cleaning

### SPRO_HEAT_ZONES
**Purpose**: Smart dual-zone bed heating based on material type  
**Parameters**:
- `MATERIAL`: Material type (PLA, PETG, ABS, ASA, TPU, PC)
**Features**:
- Automatically sets appropriate inner and outer bed temperatures
- Waits for both zones to reach temperature
- Uses material-specific temperature profiles from _S1_PRO_SETTINGS

### SPRO_MESH_MANAGER
**Purpose**: Automatically load or create bed mesh for current temperature  
**Parameters**:
- `BED_TEMP`: Target bed temperature
**Process**:
1. Checks if mesh profile exists for current temperature
2. Loads existing profile if available
3. Creates and saves new profile if not found
**Naming**: Profiles saved as `s1pro_[temp]c` (e.g., s1pro_60c)

### SPRO_PRIME_LINE
**Purpose**: Draw a prime line to prepare nozzle before printing  
**Process**:
1. Moves to safe position at edge of bed (Y-145)
2. Draws line while extruding ~26mm of filament
3. Creates consistent pressure in nozzle
4. Removes any oozing or old material
**Note**: Automatically called by SPRO_PRINT_START

### WAIT_FOR
**Purpose**: Wait for specified time with progress updates  
**Parameters**:
- `WAIT_TIME`: Time to wait in seconds (default: 60)
**Features**:
- Shows remaining time every 30 seconds
- Useful for heat soaking or time-based operations

## Best Practices

1. **Always verify sensor state** before starting prints using `CHECK_SENSORS`
2. **Follow the exact filament change procedure** - the filament guide tube connector must be opened
3. **Don't skip the 45-degree cut** - ensures smooth filament insertion
4. **Re-enable sensors** after any filament operation before printing
5. **Use material-specific macros** for optimal results
6. **Monitor first layer** after material changes due to Z-offset differences

## Troubleshooting

### Filament Won't Load
1. Ensure filament guide tube connector is closed
2. Check filament is cut at 45-degree angle
3. Verify filament reached extruder gears
4. Confirm correct temperature for material

### Screen Stuck During Load/Unload
If the printer screen animation doesn't complete:
1. The macro properly disables the extruder motor to signal completion
2. Check console for any error messages
3. Verify the macro completed (look for "Load complete" message)
4. If stuck, you can manually run `SET_STEPPER_ENABLE STEPPER=extruder ENABLE=0`

### Interface Locked/Unresponsive
If the web interface becomes unresponsive or "Unlock" button doesn't work:
1. **Common with embedded browsers** - OrcaSlicer, PrusaSlicer built-in browsers often have issues
2. **Use external browser** - Open Fluidd/Mainsail in Chrome, Firefox, or Safari
3. **Clear browser cache** - Force refresh with Ctrl+F5 (Cmd+Shift+R on Mac)
4. **Check resources** - Run `free -h` in SSH to check if system is low on memory
5. **Restart interface** - `sudo service nginx restart` or `sudo service moonraker restart`
6. **Last resort** - Restart Klipper: `sudo service klipper restart`

### False Sensor Triggers
1. Run `CHECK_SENSORS` to verify state
2. Ensure sensors were disabled during last filament change
3. Check for partial clogs causing inconsistent movement

### Filament Guide Tube Issues
- If resistance is felt during manual push, check for:
  - Kinks in filament guide tube
  - Debris in tube
  - Incorrect tube routing
  - Worn tube requiring replacement
- Note: Unlike bowden systems, the guide tube doesn't affect retraction settings

## Advanced Features

### Exclude Object Support
The printer now supports excluding individual objects during multi-part prints:
- **Enable in Slicer**: Turn on "Label objects" in PrusaSlicer/OrcaSlicer
- **During Print**: Use the web interface to exclude failed objects
- **G-code Commands**:
  - `EXCLUDE_OBJECT NAME=object_name` - Exclude specific object
  - `EXCLUDE_OBJECT_DEFINE` - Define objects (handled by slicer)
- **Benefits**: Save time and material when one object fails in a multi-part print

### Enhanced Filament Change (M600)
Improved M600 implementation with proper delta positioning:
- **Usage**: `M600` or `M600 X=0 Y=-140 Z=10`
- **Features**:
  - Saves current position and state
  - Moves to safe change position
  - Performs enhanced unload with tip forming
  - Waits for user to load new filament
  - Restores position after change

### TMC5160 Advanced Features
The extruder motor now uses advanced TMC5160 features:
- **StealthChop**: Ultra-quiet operation
- **CoolStep**: Automatic current reduction for efficiency
- **Interpolation**: 256 microstep interpolation for smooth motion

### Thermal Protection
Enhanced thermal runaway protection for all heaters:
- Monitors temperature rise rates
- Prevents runaway conditions
- Automatic shutdown on anomalies

## Safety Notes

- Never force filament if resistance is encountered
- Always wait for proper temperature before load/unload
- The delta printer has limited safe parking areas - macros use tested positions
- Dual-zone bed requires both zones to reach temperature for optimal adhesion

## Configuration Changes (2025-07-20)

### Critical Issues Resolved

#### 1. **Pressure Advance Configuration** (printer.cfg)
- **Issue**: Configuration was set for bowden system (PA=0.6) when the printer has a Direct Drive Extruder
- **Fix**: Set pressure advance to 0.05 (optimal for direct drive systems)
- **Why**: Direct drive systems require much lower PA values (0.02-0.08) compared to bowden systems (0.5-1.0)

#### 2. **Power Loss Recovery** (flsun_func.cfg)
- **Issue**: `G4 P5000` (5-second wait) commands after SHUTDOWN would never execute
- **Fix**: Removed all wait commands after SHUTDOWN
- **Why**: System cannot execute commands after shutdown, making recovery impossible

#### 3. **relay_off Macro** (flsun_func.cfg)
- **Issue**: Incorrectly disabled extruder stepper when turning off relay
- **Fix**: Removed `SET_STEPPER_ENABLE STEPPER=extruder ENABLE=0` from relay_off
- **Why**: This could cause missed steps or position loss during prints

#### 4. **Motion Sensor Detection** (flsun_func.cfg)
- **Issue**: Detection length was set for bowden system characteristics
- **Fix**: Set to 10mm for direct drive operation
- **Why**: Direct drive systems have immediate response and lower detection length prevents false triggers

#### 5. **Z-Offset Range** (printer.cfg)
- **Issue**: Limited to ±0.1mm, too restrictive for different surfaces
- **Fix**: Increased to ±0.2mm range
- **Why**: Different build surfaces and materials need more adjustment range

### New Features Added

#### 1. **Dual-Zone Bed Control** (s1_pro_macros.cfg)
- Added `M141` command to set outer bed zone temperature
- Added `M191` command to wait for outer bed zone
- Added `BED_TEMPS` macro for easy dual-zone control
- Modified heating routines to properly use both zones

#### 2. **Fan Power Control** (printer.cfg)
- Removed 55% power limitation on part cooling fan
- Now allows full power control via slicer or macros

#### 3. **Legacy Macro Handling** (printer.cfg)
- Converted old bed_level_1/2 and LOAD/UNLOAD_FILAMENT to aliases
- Now redirect to enhanced SPRO versions with proper sensor management

### Technical Details

#### Klipper Dictionary Syntax Fix (s1_pro_macros.cfg)
- Converted multi-line dictionary definitions to single-line format
- Required by Klipper's configuration parser

#### LED Configuration Cleanup (flsun_func.cfg)
- Removed all "temporary change" comments from 2024
- Made white LED configuration permanent

### Recommendations After Update

1. **Restart Klipper** to apply all configuration changes
2. **Calibrate Pressure Advance** using:
   ```
   TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0.02 FACTOR=.005
   ```
3. **Test Dual-Zone Heating**:
   ```
   BED_TEMPS INNER=60 OUTER=55
   ```
4. **Verify Sensors**:
   ```
   CHECK_SENSORS
   ```

### Why These Changes Matter

1. **Print Quality**: Proper pressure advance eliminates blobbing and stringing
2. **Reliability**: Fixed power loss recovery ensures prints can resume after outages
3. **Safety**: Relay fix prevents unexpected extruder behavior
4. **Functionality**: Dual-zone heating provides better first layer adhesion
5. **Usability**: Clearer macro organization reduces confusion