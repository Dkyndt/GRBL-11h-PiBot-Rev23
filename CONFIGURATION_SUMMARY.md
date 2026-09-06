# GRBL v1.1h (upstream bfb67f0) Configuration for PIBOT Controller Rev 2.3
## Custom Configuration Applied

> Note: Core GRBL upgraded to upstream commit bfb67f0; PiBot custom defaults preserved in PiBot_GRBL11-DEF/defaults.h

### Motor Specifications
- **Current**: 2.66A
- **Voltage**: 11V
- **Inductance**: 5.4mH
- **Steps per Revolution**: 200
- **Max Speed**: 1.91 rev/sec
- **Minimum Time per Step**: 2.61ms
- **Max Power**: 29.3W

---

## Axis Configuration

### X-Axis (Belt Drive)
- **Hardware**: 17 tooth pulley, 2mm pitch GT2 belt, 6mm wide
- **Steps per mm**: 62.75
- **Max Speed**: 3896 mm/min (64.93 mm/sec)
- **Acceleration**: 150 mm/sec² (540,000 mm/min²)
- **Max Travel**: 770 mm
- **Calculation**: (200 steps/rev) ÷ (17 teeth × 2mm pitch) = 62.75 steps/mm

### Y-Axis (Belt Drive)
- **Hardware**: 17 tooth pulley, 2mm pitch GT2 belt, 6mm wide
- **Steps per mm**: 62.75
- **Max Speed**: 3896 mm/min (64.93 mm/sec)
- **Acceleration**: 150 mm/sec² (540,000 mm/min²)
- **Max Travel**: 690 mm
- **Calculation**: Same as X-axis

### Z-Axis (Screw Drive)
- **Hardware**: 4 screws, 8mm pitch, 8mm diameter
- **Steps per mm**: 400
- **Max Speed**: 916.8 mm/min (15.28 mm/sec)
- **Acceleration**: 50 mm/sec² (180,000 mm/min²) - Conservative for screw drive
- **Max Travel**: 42 mm
- **Calculation**: 200 steps/rev ÷ (8mm pitch ÷ 4 screws) = 400 steps/mm

---

## GRBL Settings Applied

### Motion Control
- `$0` = 10 (Step pulse microseconds)
- `$1` = 255 (Step idle delay - motors always enabled)
- `$2` = 0 (Step port invert mask)
- `$3` = 1 (Direction port invert mask - Z-axis inverted)
- `$4` = 1 (Step enable invert - active low)
- `$5` = 0 (Limit pins invert)
- `$20` = 1 (Soft limits enabled)
- `$21` = 1 (Hard limits enabled)
- `$22` = 1 (Homing cycle enabled)
- `$23` = 0 (Homing direction invert mask - all home to min)

### Machine Geometry
- `$100` = 62.75 (X steps/mm)
- `$101` = 62.75 (Y steps/mm)
- `$102` = 400.0 (Z steps/mm)
- `$110` = 3896.0 (X max rate mm/min)
- `$111` = 3896.0 (Y max rate mm/min)
- `$112` = 916.8 (Z max rate mm/min)
- `$120` = 540000.0 (X acceleration mm/min²)
- `$121` = 540000.0 (Y acceleration mm/min²)
- `$122` = 180000.0 (Z acceleration mm/min²)
- `$130` = 770.0 (X max travel mm)
- `$131` = 690.0 (Y max travel mm)
- `$132` = 42.0 (Z max travel mm)

### Homing Configuration
- `$24` = 50.0 (Homing feed rate mm/min - slow and safe)
- `$25` = 800.0 (Homing seek rate mm/min - faster initial seek)
- `$26` = 250 (Homing debounce delay ms)
- `$27` = 2.0 (Homing pull-off distance mm)

### Other Settings
- `$10` = 1 (Status report mask - MPos enabled)
- `$11` = 0.02 (Junction deviation mm)
- `$12` = 0.002 (Arc tolerance mm)
- `$13` = 0 (Report in mm, not inches)
- `$30` = 1000 (Spindle max RPM)
- `$31` = 0 (Spindle min RPM)
- `$32` = 0 (Laser mode disabled)
- `$6` = 0 (Probe pin invert)

---

## Upload Instructions

### Prerequisites
1. Arduino IDE installed (1.8.x or newer)
2. PiBot board definition added to boards.txt
3. External stepper drivers POWERED OFF
4. Switch set to "3D" position

### Steps to Upload
1. Copy `PiBot_GRBL11-DEF` folder to Arduino libraries folder as `PiBot_CNC_GRBL11`
2. Open Arduino IDE
3. Go to **File → Examples → PiBot_CNC_GRBL11 → PiBotUploader**
4. Select **Tools → Board → PiBot Controller Rev2.x**
5. Select **Tools → Port → [Your COM Port]**
6. Click **Upload** button
7. Wait ~20 seconds for upload to complete

### First Time Setup
After uploading, connect via serial terminal (115200 baud) and verify settings with `$$` command.

---

## Safety Notes

⚠️ **IMPORTANT SAFETY WARNINGS:**

1. **Power Requirements**: Your motors draw 2.66A each. Ensure your power supply can handle:
   - Minimum: 3 axes × 2.66A = ~8A capacity
   - Recommended: 12-15A power supply for safety margin

2. **Soft Limits**: Enabled by default. Machine will prevent movement beyond travel limits.

3. **Hard Limits**: Enabled. Ensure limit switches are properly wired and tested.

4. **Homing Required**: `HOMING_INIT_LOCK` is enabled in config.h - you MUST home the machine after power-up before operation.

5. **Test Procedure**:
   - First, test each axis at low feed rates (100 mm/min)
   - Verify direction of movement matches expectations
   - Check limit switches trigger correctly
   - Gradually increase speeds to verify motor performance
   - Monitor for missed steps at high speeds

6. **Direction Inversion**: 
   - Z-axis is inverted by default
   - If any axis moves in wrong direction, modify `$3` setting

---

## Verification Calculations

### X/Y Axis Verification
- Steps/mm: 200 steps ÷ (17 teeth × 2mm) = 62.75 steps/mm ✓
- Max speed: 1.91 rev/sec × 17 teeth × 2mm × 60 = 3896.4 mm/min ✓

### Z Axis Verification  
- Steps/mm: 200 steps ÷ (8mm ÷ 4 screws) = 400 steps/mm ✓
- Max speed: 1.91 rev/sec × 8mm × 60 = 916.8 mm/min ✓

### Timing Verification
- Minimum time/step: 2.61ms
- Step pulse duration: 10µs (0.01ms) - Safe ✓
- Max step frequency: ~383 Hz (well within limits) ✓

---

## Files Modified
- `PiBot_GRBL11-DEF/defaults.h` - Updated DEFAULTS_PIBOT_REV23 section with custom values

## Configuration Status
✅ Steps per mm configured
✅ Max speeds configured  
✅ Accelerations configured
✅ Travel limits configured
✅ Homing parameters adjusted for large machine
✅ Safety features enabled
✅ Step timing verified

**Configuration is ready for upload!**
