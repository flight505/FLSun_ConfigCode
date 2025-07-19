# FLSUN S1 PRO Klipper Configuration

This repository contains Klipper configuration files and custom macros for the FLSUN S1 PRO delta 3D printer.

## Important Printer-Specific Information

### Filament Path and Sensors
The FLSUN S1 PRO uses a dual-sensor system with a long bowden tube setup:
- **Filament Sensor Location**: Mounted near the filament spool holder at the top of the printer
- **Motion Sensor**: Detects filament movement to identify clogs or grinding
- **Switch Sensor**: Changes state when filament passes through
- **Bowden Tube**: Approximately 700-800mm long from sensor to extruder

### Build Specifications
- **Build Plate**: Textured PEI plate with maximum temperature of 120°C
- **Build Volume**: Φ320mm × 430mm (cylinder with 383mm maximum height)
- **Heating System**: Smart zone heating system for precise temperature control and improved print quality

### Critical Operating Procedures

#### To Unload Filament:
1. Heat the hotend to appropriate temperature
2. Open the bowden connector located 10cm above the extruder
3. Run `SMART_UNLOAD` macro
4. Pull the filament out of the extruder when prompted
5. Close the bowden connector

#### To Change Filament:
1. Open the bowden connector 10cm above the extruder
2. Run `SMART_UNLOAD` macro and pull filament out
3. Cut the extracted filament at a 45-degree angle
4. Remove the 10cm piece from the extruder and discard
5. Manually retract remaining filament by rolling the spool
6. Insert new filament into the sensor at the top
7. Push filament through the bowden tube past the connector into the extruder
8. Run `SMART_LOAD` macro
9. Close the bowden connector

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
- `EXTRUDER_TEMP` (optional): Override temperature (default: 230°C)
**Process**:
1. Heats extruder to specified temperature
2. Automatically disables sensors to prevent false triggers
3. Multi-stage loading sequence:
   - Slow initial feed (20mm @ 5mm/s)
   - Fast bowden traverse (40mm @ 25mm/s)
   - Medium approach (30mm @ 10mm/s)
   - Slow final insertion (15mm @ 2.5mm/s)
4. Prompts user to re-enable sensors after completion

### SMART_UNLOAD
**Purpose**: Safely unloads filament with proper retraction sequence  
**Parameters**:
- `EXTRUDER_TEMP` (optional): Override temperature (default: 230°C)
**Process**:
1. Heats extruder to specified temperature
2. Disables sensors automatically
3. Multi-stage unloading:
   - Slow initial retract (30mm @ 5mm/s)
   - Tip forming (10mm @ 50mm/s)
   - Medium retract (20mm @ 25mm/s)
   - Fast bowden clear (80mm @ 50mm/s)
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
2. **Follow the exact filament change procedure** - the bowden connector must be opened
3. **Don't skip the 45-degree cut** - ensures smooth filament insertion
4. **Re-enable sensors** after any filament operation before printing
5. **Use material-specific macros** for optimal results
6. **Monitor first layer** after material changes due to Z-offset differences

## Troubleshooting

### Filament Won't Load
1. Ensure bowden connector is closed
2. Check filament is cut at 45-degree angle
3. Verify filament reached extruder gears
4. Confirm correct temperature for material

### False Sensor Triggers
1. Run `CHECK_SENSORS` to verify state
2. Ensure sensors were disabled during last filament change
3. Check for partial clogs causing inconsistent movement

### Bowden Tube Issues
- If resistance is felt during manual push, check for:
  - Kinks in bowden tube
  - Debris in tube
  - Incorrect tube routing
  - Worn tube requiring replacement

## Safety Notes

- Never force filament if resistance is encountered
- Always wait for proper temperature before load/unload
- The delta printer has limited safe parking areas - macros use tested positions
- Dual-zone bed requires both zones to reach temperature for optimal adhesion