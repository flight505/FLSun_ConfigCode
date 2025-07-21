# FLSUN S1 PRO Klipper Configuration Troubleshooting Guide

## Common Issues and Solutions

### 1. "Unknown command: S1" Error

**Problem**: When using macros that start with "S1_PRO", Klipper reports "Unknown command: S1"

**Cause**: Klipper's G-code parser has specific naming rules for macros. When a macro name contains numbers, they must ALL be at the end of the name. The parser interprets "S1" at the beginning as a traditional G-code command (single letter + number), which doesn't exist.

**Solution**: All S1_PRO macros have been renamed to SPRO to avoid this parser issue:
- `S1_PRO_PRINT_START` → `SPRO_PRINT_START`
- `S1_PRO_PRINT_END` → `SPRO_PRINT_END`
- `S1_PRO_HEAT_ZONES` → `SPRO_HEAT_ZONES`
- `S1_PRO_MATERIAL_SETTINGS` → `SPRO_MATERIAL_SETTINGS`
- `S1_PRO_MESH_MANAGER` → `SPRO_MESH_MANAGER`
- `S1_PRO_PRIME_LINE` → `SPRO_PRIME_LINE`
- `S1_PRO_MAINTENANCE_MODE` → `SPRO_MAINTENANCE_MODE`
- `S1_PRO_FULL_CALIBRATION` → `SPRO_FULL_CALIBRATION`

**Action Required**: Update your slicer start/end G-code to use the new macro names.

### 2. Web Interface (Fluidd/Mainsail) Freezing

**Problem**: The web interface becomes unresponsive or "locked", requiring unlock

**Common Causes**:
1. **Embedded Browser Issues**: When using Fluidd/Mainsail through embedded browsers (like in OrcaSlicer), the interface may freeze
2. **Resource Exhaustion**: Low system RAM can cause interface issues
3. **Browser Cache**: Corrupted cache data

**Solutions**:

1. **Use External Browser**: Access Fluidd/Mainsail directly through Chrome, Firefox, or Safari instead of embedded browsers

2. **Clear Browser Cache**: 
   - Press Ctrl+Shift+Delete (Cmd+Shift+Delete on Mac)
   - Clear cached images and files
   - Restart browser

3. **Check System Resources**:
   ```bash
   # SSH into your printer and check memory usage
   free -h
   # Check if any process is consuming excessive resources
   top
   ```

4. **Restart Services** (if interface remains frozen):
   ```bash
   sudo systemctl restart klipper
   sudo systemctl restart moonraker
   sudo systemctl restart nginx  # or your web server
   ```

5. **For KlipperScreen Users**:
   - In KlipperScreen settings, find 'Screen DPMS' and turn it OFF
   - Add your device IP to Moonraker's trusted clients in `moonraker.conf`

### 3. Macro Naming Best Practices

To avoid parser issues in Klipper:

**Valid Macro Names**:
- `MY_MACRO` - All letters with underscores
- `TEST_MACRO25` - Numbers only at the end
- `PRINT_ABS_245` - Numbers only at the end

**Invalid Macro Names**:
- `S1_PRO` - Number in the middle
- `MACRO25_TEST3` - Numbers in the middle
- `1ST_MACRO` - Starts with a number

### 4. Quick Diagnostics

If you encounter issues, collect these logs:
```bash
# Klipper log
cat /tmp/klippy.log

# Moonraker log
journalctl -u moonraker -n 100

# For display issues
systemctl status KlipperScreen > ~/printer_data/logs/KlipperScreen_systemctl.log
journalctl -xe -u KlipperScreen > ~/printer_data/logs/KlipperScreen_journalctl.log
```

## Need More Help?

1. Check the [Klipper Documentation](https://www.klipper3d.org/)
2. Visit the [Klipper Discord](https://discord.klipper3d.org/)
3. Post in the [Klipper Discourse Forum](https://klipper.discourse.group/)

Remember to include your `klippy.log` when asking for help!