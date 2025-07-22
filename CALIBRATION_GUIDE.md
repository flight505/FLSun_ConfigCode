# SPRO Full Calibration Guide - UPDATED

## Overview
The SPRO_FULL_CALIBRATION macro performs a complete calibration of your FLSUN S1 PRO printer in a single uninterrupted sequence. This process takes approximately 60-90 minutes.

## Latest Improvements (Fixed Version)
- Removed F104 command that was causing failures
- Added cool-down periods between PID calibrations
- Added calibration state tracking
- Improved error handling with M400 waits
- Better progress messages throughout

## Important Notes

### Why Previous Calibration Attempts Failed
1. **PID_CALIBRATE Messages**: When Klipper runs PID_CALIBRATE, it displays a message saying "The SAVE_CONFIG command will update the printer config file". This is just an informational message - it doesn't actually save or restart.

2. **Multiple SAVE_CONFIG Calls**: The original macros (PID_HOTEND, PID_BED, MEASURING_RESONANCES) each called SAVE_CONFIG at the end, causing Klipper to restart and interrupt the sequence.

3. **F104 Command**: The F104 command in MEASURING_RESONANCES was undefined and caused failures.

4. **Motor Calibration Messages**: The "please calibrate motor A/B/C!" messages are part of the normal calibration process, not errors.

5. **No Cool-down**: PID calibrations ran back-to-back without cooling, potentially affecting results.

## How the Fixed Version Works

### New Macros Created
- **SPRO_PID_HOTEND**: Performs hotend PID without SAVE_CONFIG
- **SPRO_PID_BED**: Performs bed PID (both zones) without SAVE_CONFIG  
- **SPRO_MEASURING_RESONANCES**: Performs resonance testing without SAVE_CONFIG
- **SPRO_MOTOR_CALIBRATION**: Cleaner motor calibration with better messages

### Calibration Stages
1. **Stage 1: Motor Calibration** (~1 minute)
   - Calibrates all three delta motors
   - Shows progress messages

2. **Stage 2: Hotend PID** (~5 minutes)
   - Tunes PID at 240°C
   - Calculates but doesn't save values

3. **Stage 3: Bed PID** (~10-15 minutes)
   - Tunes inner bed zone at 60°C
   - Tunes outer bed zone at 60°C
   - Calculates but doesn't save values

4. **Stage 4: Input Shaper** (~5 minutes)
   - Measures resonances with ADXL345
   - Calculates optimal shaper values

5. **Stage 5: Delta Calibration** (~5 minutes)
   - Calibrates delta geometry
   - Probes multiple points

6. **Stage 6: Bed Meshes** (~20-25 minutes)
   - Creates meshes at 60°C, 70°C, 80°C, 90°C, 100°C
   - Each includes 2-minute heat soak

### Final Save
Only after ALL calibrations complete does the macro call SAVE_CONFIG once, saving all results and restarting Klipper.

## Usage

### Running Full Calibration
```
FIRMWARE_RESTART
SPRO_FULL_CALIBRATION
```

### Monitoring Progress
In another terminal or browser tab:
```
CHECK_CALIBRATION_STATUS
```

### If Calibration Gets Stuck
```
RESET_CALIBRATION_STATE
FIRMWARE_RESTART
```

### Testing Without Full Calibration
To verify the PID calibration doesn't cause restarts:
```
TEST_PID_NO_SAVE
```

### If Calibration Fails
If the calibration stops partway through:

1. Check the console for error messages
2. Run individual calibrations:
   ```
   # For PID calibrations (these WILL restart Klipper):
   PID_HOTEND HOTEND_TEMP=240
   PID_BED BED_TEMP=60
   
   # For other calibrations:
   CALIBRATE_MOTOR
   MEASURING_RESONANCES
   DELTA_CALIBRATE
   BED_MESH_CALIBRATE
   ```

## What to Expect

### Console Messages
- Stage markers show progress (Stage 1/6, 2/6, etc.)
- PID_CALIBRATE will show "The SAVE_CONFIG command will update..." - this is normal
- Motor calibration shows "please calibrate motor A/B/C!" - this is normal
- Cool-down messages appear between PID calibrations
- Each stage saves its progress to variables
- Final message confirms all data is being saved

### Temperature Readings
- During PID calibration, temperatures will cycle up and down
- This is the tuning algorithm finding optimal values
- Fan will run at 100% during PID tuning

### Final Result
After successful completion:
- Klipper will restart automatically
- All calibration values will be saved to printer.cfg
- You'll see new values in the SAVE_CONFIG section

## Troubleshooting

### Calibration Stops After Hotend PID
- Check if Klipper restarted (look for "Variable Saved:plr_flag = False")
- Verify you're using SPRO_FULL_CALIBRATION, not individual macros
- Check /tmp/klippy.log for errors

### "Unknown Command" Errors
- Make sure you've restarted Klipper after adding new macros
- Verify the macros were saved correctly

### Temperature Not Reaching Target
- Check heater connections
- Verify thermistor readings
- Ensure power supply is adequate

## New Utility Macros

### CHECK_CALIBRATION_STATUS
Shows which stage of calibration is currently running:
```
CHECK_CALIBRATION_STATUS
```

### RESET_CALIBRATION_STATE  
Clears calibration tracking if it gets stuck:
```
RESET_CALIBRATION_STATE
```

### RUN_INDIVIDUAL_CALIBRATIONS
Provides instructions for running each calibration manually:
```
RUN_INDIVIDUAL_CALIBRATIONS
```

## Next Steps
After successful calibration:
1. Run a test print to verify results
2. Fine-tune pressure advance if needed
3. Adjust Z-offset for your specific build surface
4. Consider the probe accuracy improvements suggested by the community