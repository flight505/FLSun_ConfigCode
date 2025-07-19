# Changelog

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