# Changelog

## [2025-07-20] - Critical Update: Direct Drive Extruder Configuration

### Summary
Major configuration overhaul after discovering the FLSUN S1 PRO uses a Direct Drive Extruder (DDE), not a bowden system. The long PTFE tube is only a filament guide from the dryer to the direct drive extruder. All settings have been updated to reflect proper direct drive characteristics. Additional fixes for temperature management and macro messages.

---

## Files Changed

### 1. **printer.cfg** (MODIFIED)

#### Critical Changes:
- **pressure_advance**: Changed from 0.6 (bowden) to 0.05 (direct drive)
- **pressure_advance_smooth_time**: Added 0.040 for direct drive smoothing
- **max_extrude_only_distance**: Reduced from 800 to 100 (standard for direct drive)

#### Why These Changes:
- Direct drive extruders require pressure advance values between 0.02-0.08
- The 700-800mm tube is just a guide, not a bowden tube affecting extrusion
- Direct drive only needs ~100mm max extrude distance for filament changes

#### CLEAN_FILAMENT Macro Fix:
- **Issue**: Displayed "unload_filament done!" after extruding 160mm (confusing message)
- **Fix**: Changed messages to "Heating for filament purge..." and "Filament purge complete!"
- **Added**: `M104 S0` to turn off hotend after purging (safety improvement)
- **Purpose**: Now clearly indicates it's for purging/cleaning, not unloading

---

### 2. **s1_pro_macros.cfg** (MODIFIED)

#### SMART_LOAD Changes:
- Reduced total load distance from 600mm to 30mm
- Simplified from 5 stages to 3 stages:
  - Stage 1: Initial feed (10mm @ 5mm/s)
  - Stage 2: Prime extruder (15mm @ 10mm/s)  
  - Stage 3: Final extrusion (5mm @ 2.5mm/s)

#### SMART_UNLOAD Changes:
- Reduced total unload distance from 560mm to 30.5mm
- New retraction sequence for direct drive:
  - Stage 1: Retract from nozzle (0.5mm @ 35mm/s)
  - Stage 2: Clear hotend (15mm @ 25mm/s)
  - Stage 3: Final retract (15mm @ 50mm/s)

#### Other Changes:
- Updated message from "Open bowden coupling" to "Remove filament from guide tube"
- **FIXED**: Screen animation blocking issue in SMART_LOAD:
  - Added `SET_STEPPER_ENABLE STEPPER=extruder ENABLE=0` to properly signal completion
  - Removed `M104 S0` to maintain temperature for subsequent operations
  - Screen now properly completes loading animation and becomes responsive

#### S1_PRO_FULL_CALIBRATION Enhanced:
- **Previous**: Only performed delta calibration and bed meshes
- **Now includes**: Complete calibration sequence:
  1. Motor calibration (CALIBRATE_MOTOR)
  2. Hotend PID tuning at 240°C
  3. Dual-zone bed PID tuning at 60°C
  4. Input shaper/resonance calibration
  5. Delta calibration
  6. Bed mesh profiles for multiple temperatures
- **Duration**: ~45-60 minutes (was ~20-30 minutes)
- **Purpose**: True one-command complete system calibration

---

### 3. **flsun_func.cfg** (MODIFIED)

#### Motion Sensor Changes:
- **detection_length**: Reduced from 25.0mm to 10.0mm
- Prevents false triggers common with direct drive's immediate response
- Better suited for direct drive extrusion characteristics

---

### 4. **README.md** (MODIFIED)

#### Major Updates:
- Added "with Direct Drive Extruder (DDE)" to main description
- Clarified the 700-800mm tube is a "Filament Guide Tube" not a bowden tube
- Added explanation of Direct Drive Extruder mounting
- Updated all pressure advance references for direct drive values
- Changed calibration command from START=0.4 to START=0.02
- Updated all loading/unloading sequences for direct drive distances
- Replaced all "bowden" references with "filament guide tube"
- Added note that guide tube doesn't affect retraction settings

---

## Technical Improvements

### 1. **Direct Drive Optimization**
- Properly configured pressure advance for immediate filament response
- Retraction settings now match direct drive characteristics (0.5mm vs 500mm)
- Loading/unloading sequences optimized for short filament path

### 2. **Corrected Misconceptions**
- Clarified that the long PTFE tube is just a guide, not affecting extrusion
- Updated all documentation to reflect actual hardware configuration
- Removed bowden-specific compensations that were causing issues

### 3. **Expected Benefits**
- Better print quality with proper pressure advance
- Reduced stringing with appropriate retraction
- More accurate filament sensor detection
- Improved response to filament changes
- Better handling of flexible filaments
- Hotend properly turns off after loading filament (safety improvement)

---

## Migration Notes

Users updating from the previous configuration should:
1. Re-run pressure advance calibration starting at 0.02
2. Update slicer retraction settings to 0.5-1.0mm
3. Verify filament sensor sensitivity
4. Test new load/unload sequences with each material type

---

## [2025-07-19] - FLSUN S1 PRO Configuration Updates

### Summary
Major updates to improve filament handling for the FLSUN S1 PRO's long bowden tube system, comprehensive documentation, and enhanced macro functionality.

---

## Files Changed

### 1. **README.md** (NEW FILE)
Created comprehensive documentation for the FLSUN S1 PRO configuration.

#### Added sections:
- **Important Printer-Specific Information**
  - Detailed explanation of dual sensor system
  - Critical operating procedures for filament changes
  - Why sensor disabling is required during load/unload
  
- **Macro Overview**
  - Categorized list of all macros by function
  
- **Detailed Macro Descriptions**
  - Full documentation for each macro including:
    - Purpose and functionality
    - Parameters with defaults
    - Step-by-step process
    - Usage notes
    
- **Best Practices**
  - Operational guidelines
  - Safety considerations
  
- **Troubleshooting**
  - Common issues and solutions
  - Bowden tube specific problems

---

### 2. **CLAUDE.md** (NEW FILE)
Created guidance file for Claude Code AI assistant.

#### Added:
- Repository context and purpose
- Key files and their functions
- Important technical context (delta kinematics, dual sensors, etc.)
- Common development tasks
- Architecture patterns
- Critical safety considerations

---

### 3. **s1_pro_macros.cfg** (MODIFIED)

#### Modified Macros:

##### SMART_LOAD
**Changes:**
- Increased total load distance from ~105mm to 600mm
- Updated loading stages from 4 to 5 for better control:
  - Stage 1: 20mm → 50mm (initial feed)
  - Stage 2: 40mm → 300mm (fast bowden traverse)
  - Stage 3: 30mm → 200mm (continue through bowden)
  - Stage 4: 30mm → 40mm (approaching hotend)
  - Stage 5: 15mm → 10mm (final extrusion)
- Updated stage descriptions for clarity
- Adjusted feed rates for optimal performance

##### SMART_UNLOAD  
**Changes:**
- Added TEMP parameter support: `{% set extruder_temp = params.TEMP|default(235)|int %}`
- Added sensor disabling at start of unload process:
  ```
  SET_FILAMENT_SENSOR SENSOR=filament_sensor ENABLE=0
  SET_FILAMENT_SENSOR SENSOR=my_sensor ENABLE=0
  RESPOND MSG="⚠️ Sensors temporarily disabled for unloading process"
  ```
- Completely redesigned unload sequence for bowden system:
  - Old: 30mm + 10mm + 20mm + 80mm = 140mm total
  - New: 10mm push + 10mm retract + 50mm + 500mm = 560mm total
- Changed from 4 stages to 3 optimized stages:
  - Stage 1: Tip forming (push forward and quick retract)
  - Stage 2: Clear hotend (50mm fast retract)
  - Stage 3: Fast bowden retract (500mm)
- Added reminder to re-enable sensors after unload

##### UNLOAD (Quick Access Macro)
**Changes:**
- Added TEMP parameter pass-through: `SMART_UNLOAD TEMP={params.TEMP|default(235)}`
- Previously had no temperature parameter support

#### Added Macros:

##### Material-Specific Load/Unload Macros (12 new macros)
Added dedicated macros for each supported material:
- `LOAD_PLA` / `UNLOAD_PLA` (210°C)
- `LOAD_PETG` / `UNLOAD_PETG` (240°C)
- `LOAD_ABS` / `UNLOAD_ABS` (260°C)
- `LOAD_ASA` / `UNLOAD_ASA` (270°C)
- `LOAD_TPU` / `UNLOAD_TPU` (220°C)
- `LOAD_PC` / `UNLOAD_PC` (300°C)

Each macro calls SMART_LOAD or SMART_UNLOAD with the optimal temperature for that material.

---

### 4. **printer.cfg** (MODIFIED)

#### Changed:
- **max_extrude_only_distance**: 
  - Old value: `500`
  - New value: `800  # Increased for S1 PRO long bowden tube`
  - Reason: The 600mm total load distance exceeds the previous 500mm limit

---

## Technical Improvements

### 1. **Bowden Tube Optimization**
- Load/unload distances now properly account for ~700-800mm bowden tube length
- Multi-stage approach prevents jams and ensures reliable operation
- Tip forming sequence in unload prevents stringing

### 2. **Sensor Management**
- Consistent sensor disabling in both load and unload operations
- Clear user notifications about sensor state
- Reminders to re-enable sensors before printing

### 3. **Temperature Control**
- All material-specific macros use optimal temperatures
- TEMP parameter support added to all relevant macros
- Maintains temperature during paused unload operations

### 4. **User Experience**
- Clear stage-by-stage progress messages
- Descriptive error messages
- Comprehensive documentation for all procedures

---

## Breaking Changes
None - all changes are backward compatible. Existing G-code and macros will continue to function.

---

## Recommendations for Users
1. Update your slicer profiles to use material-specific load/unload macros
2. Always follow the documented filament change procedure
3. Ensure sensors are re-enabled before starting prints
4. Consider increasing retraction settings if using the old values
5. Test load/unload with each material type before production use