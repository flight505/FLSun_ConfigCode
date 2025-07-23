# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository. 

## Repository Overview
This is a Klipper configuration repository for the FLSUN S1 PRO delta 3D printer. It contains configuration files and custom macros for printer control, not traditional source code.

## Key Files and Their Purpose
- **printer.cfg** - Main Klipper configuration defining all hardware components (steppers, heaters, sensors, kinematics)
- **flsun_func.cfg** - FLSUN-specific hardware control functions (LEDs, PID tuning, power loss recovery)
- **s1_pro_macros.cfg** - Enhanced printer macros for filament handling, bed leveling, and material profiles
- **moonraker.conf** - Web API interface configuration for remote printer control

## Important Technical Context
- **Delta Kinematics**: This printer uses delta (triangular) movement, not Cartesian. Position calculations and movements are different from typical 3D printers.
- **Dual Filament Sensors**: System uses both runout detection and motion/clog detection sensors that must be managed together
- **Dual-Zone Heated Bed**: Inner and outer heating zones are controlled separately for optimal temperature distribution
- **TMC5160 Stepper Driver**: Advanced driver on extruder requires specific configuration parameters
- **Active Cooling System**: Chamber fan (box_fan) is used during calibration cool-down phases to reduce wait times
- **Probe Accuracy**: Configured for 9x9 mesh grid with 3 samples per point at reduced speed, using 0.05 tolerance to account for thermal expansion
- **Calibration Order**: Delta calibration MUST be performed before bed mesh on delta printers

## Common Development Tasks
When modifying configurations:
1. **Validate syntax**: Klipper will validate on restart - check `/tmp/klippy.log` for errors
2. **Test macros**: Use Mainsail/Fluidd console to test individual macros before full integration
3. **Backup before changes**: Critical to preserve working configurations

## Architecture Patterns
1. **Macro Organization**: 
   - Material-specific settings use standardized naming: `LOAD_<MATERIAL>`, `UNLOAD_<MATERIAL>`
   - All macros preserve printer state (temperatures, sensors) when possible
   - Error handling includes sensor state restoration

2. **Sensor Management**:
   - Filament sensors must be disabled during load/unload operations
   - Always restore original sensor states after operations
   - Both sensors (runout and motion) must be handled as a pair

3. **Temperature Handling**:
   - Macros wait for temperature stability before proceeding
   - Material profiles include both hotend and bed temperatures
   - Bed leveling preserves current temperatures unless overridden

## Critical Safety Considerations
- Power loss recovery system must not interfere with normal operations
- Filament sensor states must always be restored after macro completion
- Temperature limits are hardware-enforced but should be respected in macros
- Delta printer homing must complete before any movement commands

## Calibration Best Practices
- **Order Matters**: On delta printers, always run delta calibration before bed mesh
- **Active Cooling**: Use SPRO_ACTIVE_COOLDOWN or chamber fan during cool-down phases
- **Probe Settings**: Higher sample count and slower speed improve mesh accuracy
- **Full Calibration**: SPRO_FULL_CALIBRATION runs all calibrations in optimal order (~65 minutes)