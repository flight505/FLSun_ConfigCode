# FLSUN S1 PRO Klipper Configuration

This repository contains Klipper configuration files and custom macros for the FLSUN S1 PRO delta 3D printer with Direct Drive Extruder (DDE).

## ⚡ Recent Configuration Improvements (2025-07-20)

Major configuration improvements have been implemented to resolve critical issues and optimize performance:

### 🔧 Critical Fixes Applied:
1. **Pressure Advance Optimization** - Updated pressure advance for direct drive extruder (was incorrectly configured for bowden)
2. **Dual-Zone Bed Heating** - Fully implemented independent control of inner and outer heating zones
3. **Power Loss Recovery** - Fixed shutdown sequence that prevented proper recovery
4. **Sensor Detection** - Optimized motion sensor sensitivity for direct drive system
5. **Safety Improvements** - Resolved relay_off macro that was incorrectly disabling the extruder

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

The `s1_pro_macros.cfg` file contains comprehensive macros organized into the following categories:

### 1. **Configuration and Settings**
- `_S1_PRO_SETTINGS` - Central configuration for all macro parameters

### 2. **Filament Management**
- `SMART_LOAD` - Intelligent filament loading with sensor management
- `SMART_UNLOAD` - Safe filament unloading procedure
- `LOAD_[MATERIAL]` - Material-specific loading (PLA, PETG, ABS, ASA, TPU, PC)
- `UNLOAD_[MATERIAL]` - Material-specific unloading

### 3. **Sensor Control**
- `ENABLE_SENSORS` - Enable both filament sensors
- `DISABLE_SENSORS` - Disable both sensors for maintenance
- `CHECK_SENSORS` - Diagnostic tool for sensor status

### 4. **Temperature Management**
- `S1_PRO_MATERIAL_SETTINGS` - Apply material-specific temperatures and Z-offset

### 5. **Bed Leveling**
- `S1_PRO_BED_LEVEL` - Enhanced bed leveling with temperature preservation

### 6. **Print Management**
- `S1_PRO_PRINT_START` - Comprehensive print initialization
- `S1_PRO_PRINT_END` - Safe print completion procedure

### 7. **Dual-Zone Bed Control** (NEW)
- `M141` - Set outer bed zone temperature (usage: `M141 S70`)
- `M191` - Wait for outer bed zone to reach temperature
- `BED_TEMPS` - Set both zones at once (usage: `BED_TEMPS INNER=60 OUTER=55`)

## Detailed Macro Descriptions

### _S1_PRO_SETTINGS
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
4. Prompts user to re-enable sensors after completion

### SMART_UNLOAD
**Purpose**: Safely unloads filament with proper retraction sequence  
**Parameters**:
- `TEMP` (optional): Override temperature (default: 235°C)
**Process**:
1. Heats extruder to specified temperature
2. Disables sensors automatically
3. Multi-stage unloading optimized for direct drive:
   - Retract from nozzle (0.5mm @ 35mm/s)
   - Clear hotend (15mm @ 25mm/s)
   - Final retract (15mm @ 50mm/s)
4. Prompts for manual sensor re-enabling

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

### S1_PRO_MATERIAL_SETTINGS
**Purpose**: Applies comprehensive material-specific settings  
**Parameters**:
- `MATERIAL`: One of PLA, PETG, ABS, ASA, TPU, PC
**Settings Applied**:
- Extruder temperature
- Bed temperature (both inner and outer zones)
- Z-offset adjustment for material characteristics
**Usage**: Call before printing or when switching materials

### S1_PRO_BED_LEVEL
**Purpose**: Enhanced bed leveling routine preserving temperatures  
**Parameters**:
- `BED_TEMP` (optional): Override bed temperature
- `EXTRUDER_TEMP` (optional): Override extruder temperature  
**Features**:
- Saves current temperatures before leveling
- Performs delta calibration and bed mesh
- Restores original temperatures
- Shows bed mesh visualization

### S1_PRO_PRINT_START
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

### S1_PRO_PRINT_END
**Purpose**: Safe print completion and cooldown  
**Features**:
- Coordinated retraction
- Safe Z-lift within delta limits
- Park position at X0 Y-140
- Controlled cooldown of all heaters
- Motor idle timeout

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
- Now redirect to enhanced S1_PRO versions with proper sensor management

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