# FLSUN S1 PRO Macro Reference Guide

This guide explains the macro organization and helps you choose the right macro for your needs.

## Macro Naming Convention

We have two main macro sets:
1. **Original FLSUN macros** - Basic functionality from factory configuration
2. **SPRO_ prefixed macros** - Enhanced versions with additional features

## Which Macros to Use?

### For Automated Operations (Recommended)
Use SPRO_ versions - they include enhancements like:
- Temperature optimization
- Active cooling
- Progress tracking
- No SAVE_CONFIG calls (for chaining operations)

### For Manual Operations
Use original macros when you need immediate SAVE_CONFIG:
- `PID_HOTEND` - Manual hotend PID tuning with save
- `PID_BED` - Manual bed PID tuning with save
- `MEASURING_RESONANCES` - Manual resonance testing with save

## Calibration Macros

### Automated Full Calibration
```gcode
SPRO_FULL_CALIBRATION
```
Runs all calibrations in optimal order (~65 minutes):
1. Motor calibration
2. Delta calibration (CRITICAL: before bed mesh)
3. Hotend PID
4. Bed PID (both zones)
5. Input shaper
6. Bed mesh at multiple temperatures
7. Single SAVE_CONFIG at end

### Individual Calibrations

#### PID Tuning
- **For automation**: `SPRO_PID_HOTEND`, `SPRO_PID_BED` (no auto-save)
- **For manual use**: `PID_HOTEND`, `PID_BED` (saves immediately)

#### Resonance Testing
- **For automation**: `SPRO_MEASURING_RESONANCES` (no auto-save)
- **For manual use**: `MEASURING_RESONANCES` (saves immediately)

#### Motor Calibration
- `CALIBRATE_MOTOR` - Main user command
- `SPRO_MOTOR_CALIBRATION` - Enhanced wrapper with better messaging

## Filament Management

### Current Standard (Use These)
- `SMART_LOAD` - Intelligent loading with sensor management
- `SMART_UNLOAD` - Safe unloading with tip forming
- `LOAD_[MATERIAL]` / `UNLOAD_[MATERIAL]` - Material-specific variants

### Quick Aliases
- `LOAD` → `SMART_LOAD`
- `UNLOAD` → `SMART_UNLOAD`

### Deprecated (Don't Use)
- ~~`LOAD_FILAMENT`~~ - Redirects to SMART_LOAD
- ~~`UNLOAD_FILAMENT`~~ - Redirects to SMART_UNLOAD

## Bed Leveling

### Current Standard
- `BED_LEVEL_1` - Delta calibration
- `BED_LEVEL_2` - Bed mesh calibration
- `SPRO_BED_LEVEL` - Enhanced leveling with temperature preservation

### Legacy (Still Used Internally)
- `bed_level_1` - Lowercase version (used in power loss recovery)
- `bed_level_2` - Lowercase version (DO NOT REMOVE - used by system)

## Utility Macros

### Homing
- `_CG28` - Conditional homing (only homes if not already homed)
- `G28` - Standard homing (always homes)

### Parking
- `PARKFRONT` - Park at Y-140 (maintenance position)
- `PARKFRONTLOW` - Park at Y-140, Z=20
- `PARKCENTER` - Park at bed center
- `PARKBED` - Park 15mm above bed
- `PARKTOP` - Park at safe top position

### Temperature Control
- `M141` - Set outer bed zone temperature
- `M191` - Wait for outer bed zone
- `BED_TEMPS` - Set both bed zones
- `SPRO_ACTIVE_COOLDOWN` - Fast cooling with fans

### Debugging
- `DUMP_VARIABLES` - Display Klipper variables
- `TEST_SPEED` - Test maximum speeds/accelerations
- `GET_POSITION` - Built-in Klipper command (detailed position info)
- `CHECK_SENSORS` - Check filament sensor status

## M-Code Overrides

These standard commands have been enhanced:
- `M106` - Fan control with soft-start logic
- `M600` - Enhanced filament change procedure

## Print Start/End

Always use the SPRO versions:
- `SPRO_PRINT_START` - Comprehensive initialization
- `SPRO_PRINT_END` - Safe shutdown sequence

## Important Notes

1. **Never remove `bed_level_2`** - Used by power loss recovery system
2. **SPRO_ macros don't auto-save** - Designed for batch operations
3. **Original macros auto-save** - Good for single operations
4. **All macros work together** - Enhanced versions often call originals

## Quick Decision Guide

- **Running full calibration?** → Use `SPRO_FULL_CALIBRATION`
- **Tuning PID manually?** → Use `PID_HOTEND` or `PID_BED`
- **Loading filament?** → Use `SMART_LOAD` or `LOAD_[MATERIAL]`
- **Need to park?** → Use `PARK*` macros
- **Testing limits?** → Use `TEST_SPEED`
- **Debugging?** → Use `DUMP_VARIABLES`