# Advanced Calibration & Troubleshooting Guide
## PIBOT Controller Rev 2.3 with GRBL v1.1h (upstream bfb67f0)

> Note: Core GRBL updated to upstream commit bfb67f0; verify settings after upgrade.

---

## 📐 PRECISION CALIBRATION

### Measuring Steps/mm Accurately

Your theoretical values are:
- X/Y: 62.75 steps/mm (17T pulley, 2mm GT2 belt)
- Z: 400 steps/mm (4× 8mm pitch screws)

**Real-world factors affecting accuracy:**
1. Belt stretch and wear
2. Pulley manufacturing tolerances
3. Lead screw pitch variation
4. Microstepping accuracy
5. Mechanical backlash

### Calibration Procedure

#### X-Axis Calibration:
```bash
1. Mark start position with precision square or dial indicator
2. Set absolute positioning: G90
3. Zero the axis: G92 X0
4. Command large move: G0 X500 F1000
5. Measure actual distance traveled
6. Calculate correction:
   New_Steps = 62.75 × (500 / Actual_Distance)
7. Update: $100=New_Steps
```

**Example:**
- Commanded: 500mm
- Actual: 498.3mm
- New steps = 62.75 × (500 / 498.3) = 62.96 steps/mm
- Command: `$100=62.96`

#### Y-Axis Calibration:
Same process as X-axis, use `$101=` to update.

#### Z-Axis Calibration:
```bash
1. Place height gauge or use dial indicator
2. Set absolute positioning: G90
3. Zero the Z-axis: G92 Z0
4. Command move: G0 Z30 F500 (smaller move for Z)
5. Measure actual distance with calipers
6. Calculate: New_Steps = 400 × (30 / Actual)
7. Update: $102=New_Steps
```

**Precision Note:** Measure multiple times and average the results for best accuracy.

---

## 🔍 BELT TENSION & MECHANICAL TUNING

### Optimal Belt Tension
**Too Loose:**
- Backlash in direction changes
- Position inaccuracy
- Tooth skipping under load

**Too Tight:**
- Increased motor load
- Premature bearing wear
- Higher power consumption

**Correct Tension Test:**
- Press belt sideways at midpoint between pulleys
- Should deflect ~5-10mm with moderate finger pressure
- Pluck like guitar string - should make low "thunk" sound, not high ping

### Belt Inspection Checklist
- [ ] No visible wear or fraying
- [ ] Teeth not damaged or missing
- [ ] Belt tracks centered on pulley
- [ ] No dust or debris in teeth
- [ ] Pulleys aligned (no twist)
- [ ] Set screws tight on pulleys

### Lead Screw Maintenance (Z-Axis)
- [ ] Clean and lubricate regularly
- [ ] Check for bent or damaged threads
- [ ] Verify anti-backlash nut condition
- [ ] Check bearing alignment
- [ ] Ensure no binding through full travel

---

## ⚡ STEPPER DRIVER TUNING

### Current Setting Verification
Your motors: **2.66A rated current**

**Driver Current Formula:**
```
Vref = (Desired_Current × 8 × Rsense) / Scale_Factor
```

For A4988 drivers (common on PIBOT):
```
Vref = 2.66A × 8 × 0.05Ω = 1.06V (for full current)
Recommended: 0.85V (80% of max for cooler operation)
```

For TMC2208/2209 drivers:
```
Vref = 2.66A × 1.41 × Rsense (check your driver datasheet)
```

### Tuning Procedure:
1. Measure Vref with multimeter on driver potentiometer
2. Adjust slowly while motor is stationary
3. Test jog moves after each adjustment
4. Listen for missed steps or motor stalling
5. Feel motor temperature after 5-minute run (should be warm, not burning hot)

### Microstepping Configuration
Your configuration assumes **1/16 microstepping** (standard).

If using different microstepping:
- **1/8 step**: Divide steps/mm by 2
- **1/32 step**: Multiply steps/mm by 2

Example: X-axis with 1/32 microstepping:
```
$100=125.5  (62.75 × 2)
```

---

## 🎯 ACCELERATION TUNING

### Finding Optimal Acceleration

Current settings (conservative):
- X/Y: 540,000 mm/min² (150 mm/sec²)
- Z: 180,000 mm/min² (50 mm/sec²)

**Test Procedure:**
```gcode
G21 G90            ; mm, absolute
G0 X0 Y0 Z0        ; start position
G0 X100 Y100 F3000 ; rapid move
G0 X0 Y0           ; return
```

**Increase acceleration gradually:**
```
$120=720000  ; Try 200 mm/sec²
$121=720000
```

**Test with rapid direction changes:**
```gcode
G0 X100 F3896
G0 X0
G0 Y100
G0 Y0
G0 X100 Y100
G0 X0 Y0
```

**Signs of too much acceleration:**
- Motor stalling at start of move
- Position shift after rapid moves
- Grinding/clicking sounds
- Belt skipping teeth

**Signs you can increase more:**
- Smooth acceleration
- No position errors
- Motors not overheating
- No unusual noise

### Aggressive Settings (Test Carefully):
```
$120=900000   ; X: 250 mm/sec²
$121=900000   ; Y: 250 mm/sec²
$122=360000   ; Z: 100 mm/sec²
```

### Conservative Settings (If Having Issues):
```
$120=360000   ; X: 100 mm/sec²
$121=360000   ; Y: 100 mm/sec²
$122=180000   ; Z: 50 mm/sec²
```

---

## 🏠 HOMING OPTIMIZATION

### Homing Sequence Settings

Current configuration:
```
$22=1   ; Homing enabled
$23=0   ; All axes home to minimum (toward 0)
$24=50  ; Homing feed rate (slow final approach)
$25=800 ; Homing seek rate (fast initial search)
$26=250 ; Debounce delay (switch settling time)
$27=2.0 ; Pull-off distance after homing
```

### Customizing Homing Direction

**To home toward maximum instead of minimum:**
```
$23=1  ; X homes to max
$23=2  ; Y homes to max
$23=4  ; Z homes to max
$23=7  ; All axes home to max
```

**Mixed homing (Z to max, X/Y to min):**
```
$23=4  ; Only Z homes to max
```

### Homing Cycle Order

Edit `config.h` to change homing order (requires recompile):

```cpp
// Default: All axes home together
#define HOMING_CYCLE_0 ((1<<X_AXIS)|(1<<Y_AXIS)|(1<<Z_AXIS))

// Sequential: Z first, then X and Y
#define HOMING_CYCLE_0 (1<<Z_AXIS)
#define HOMING_CYCLE_1 ((1<<X_AXIS)|(1<<Y_AXIS))

// Completely sequential: Z, Y, X
#define HOMING_CYCLE_0 (1<<Z_AXIS)
#define HOMING_CYCLE_1 (1<<Y_AXIS)
#define HOMING_CYCLE_2 (1<<X_AXIS)
```

**Why change homing order?**
- Prevent collisions with workpieces
- Ensure Z clears before X/Y movement
- Match your machine geometry

### Limit Switch Troubleshooting

**Limit switch constantly triggered:**
```
$5=1   ; Invert limit pins
```

**Limit switch not detected:**
1. Test continuity with multimeter
2. Check wiring (should be Normally Open switches)
3. Verify 5V pullup present
4. Try different switch or adjust sensitivity

**Homing fails midway:**
1. Increase debounce: `$26=500`
2. Slow down feed rate: `$24=25`
3. Slow down seek rate: `$25=400`
4. Check switch mounting (not loose/vibrating)

---

## 🔧 ADVANCED GRBL SETTINGS

### Enable Core-XY Kinematics (If Applicable)
Edit `config.h`:
```cpp
#define COREXY  // Uncomment if using CoreXY
```

### Custom Axes (4th/5th Axis)

Your CPU map already supports A and T axes. To enable:

Edit `config.h`:
```cpp
#define N_AXIS 4  // Change from 3 to 4 for A-axis
// or
#define N_AXIS 5  // For both A and T axes
```

Add to `defaults.h` under `DEFAULTS_PIBOT_REV23`:
```cpp
#define DEFAULT_A_STEPS_PER_MM 400.0
#define DEFAULT_A_MAX_RATE 1000.0
#define DEFAULT_A_ACCELERATION (50.0*60*60)
#define DEFAULT_A_MAX_TRAVEL 360.0  // degrees
```

### Parking Motion (Tool Change)

Enable parking in `config.h`:
```cpp
#define PARKING_ENABLE
#define PARKING_AXIS Z_AXIS
#define PARKING_TARGET -5.0  // mm
#define PARKING_RATE 500.0   // mm/min
```

### Spindle PWM Configuration

For variable speed spindle (PWM control):

Check `cpu_map.h` - PIBOT uses Pin L1 for spindle enable.

Settings:
```
$30=24000  ; Max spindle RPM (set to your spindle's max)
$31=0      ; Min spindle RPM
```

Test spindle speed control:
```gcode
M3 S12000  ; Spindle CW at 12,000 RPM (50%)
M5         ; Spindle off
```

---

## 📊 PERFORMANCE BENCHMARKS

### Positioning Accuracy Test

**Create test pattern:**
```gcode
G21 G90 G0 Z5       ; Safe Z height
G0 X0 Y0            ; Origin
G0 X100 Y0          ; Point 1
G0 X100 Y100        ; Point 2
G0 X0 Y100          ; Point 3
G0 X0 Y0            ; Return to origin
```

**Measure:**
- Check all corners with precision square
- Verify 90° angles with machinist square
- Measure diagonals (should be 141.42mm for 100mm square)

**Acceptable tolerance:** ±0.1mm for well-tuned machine

### Repeatability Test

**Run 10 cycles:**
```gcode
G21 G90
G0 X0 Y0 Z0
G0 X500 Y500 F3000
G0 X0 Y0
(Repeat 10 times)
```

**Measure position after each return to origin.**
**Good result:** <0.05mm variation

### Maximum Reliable Speed Test

**Gradually increase feed rate:**
```gcode
G0 X500 F1000  ; Start
G0 X0
G0 X500 F2000  ; Increase
G0 X0
G0 X500 F3000  ; Increase
G0 X0
G0 X500 F3896  ; Maximum
G0 X0
```

**Watch for:**
- Missed steps
- Position error
- Motor stalling
- Unusual sounds

**Record maximum reliable speed** and reduce `$110`/`$111` if needed.

---

## 🛠️ COMMON MODIFICATIONS

### Increase Serial Buffer

For smoother streaming, edit `config.h`:
```cpp
#define RX_BUFFER_SIZE 256  // Default is 128
```

### Adjust Planner Blocks

For more complex paths:
```cpp
#define BLOCK_BUFFER_SIZE 32  // Default is 16
```

**Note:** Both increase RAM usage. Check compilation for memory limits.

### Enable Real-Time Overrides

Check `config.h` has these enabled (should be default):
```cpp
// Feed/Rapid/Spindle overrides already enabled in your config
```

Use during operation:
- `0x90` - Reset feed override to 100%
- `0x91` - Increase feed 10%
- `0x92` - Decrease feed 10%
- `0x93` - Increase feed 1%
- `0x94` - Decrease feed 1%

---

## 🐛 DEBUGGING TECHNIQUES

### Enable Verbose Error Messages

Connect serial terminal and type `$Errors` to see error codes.

### Monitor Real-Time Status

Send `?` command repeatedly (or use sender software with auto-status):
```
<Idle|MPos:0.000,0.000,0.000|FS:0,0>
<Run|MPos:52.361,0.000,0.000|FS:3896,0>
```

**Status codes:**
- Idle - Ready for commands
- Run - Executing G-code
- Hold - Feed hold active
- Alarm - Limit triggered or error state
- Door - Safety door open

### Log Position Drift

Create logging script:
```bash
G0 X100 Y100
?  (record position)
G0 X0 Y0
?  (record position - should be 0,0)
(Repeat 100 times and check for drift)
```

### Check EEPROM Integrity

If settings seem corrupted:
```
$RST=$  ; Reset to defaults
$$      ; Verify defaults loaded
```

Then re-apply your custom settings.

---

## 📈 OPTIMIZATION SUMMARY

**For Maximum Speed:**
- Increase accelerations cautiously
- Reduce junction deviation: `$11=0.01`
- Optimize feed rates in CAM software
- Use arcs instead of short line segments

**For Maximum Accuracy:**
- Reduce accelerations
- Increase junction deviation: `$11=0.05`
- Slow down feed rates
- Calibrate steps/mm precisely
- Reduce mechanical backlash

**For Quieter Operation:**
- Enable microstepping (hardware)
- Reduce accelerations
- Use slower feed rates during rapids
- Check for mechanical resonance frequencies

---

## 🎓 LEARNING RESOURCES

### Essential Reading
1. GRBL Wiki: https://github.com/gnea/grbl/wiki
2. G-Code Reference: https://linuxcnc.org/docs/html/gcode.html
3. CNC Cookbook Feeds & Speeds: https://www.cnccookbook.com/

### Useful Tools
- Universal G-Code Sender: https://winder.github.io/ugs_website/
- bCNC: https://github.com/vlachoudis/bCNC
- GRBLweb: https://github.com/andrewhodel/grblweb

### Testing Patterns
- Test your machine with these common patterns:
  - Concentric squares (mechanical accuracy)
  - Circle interpolation (smooth motion)
  - Rapid direction changes (acceleration)
  - Long diagonal moves (belt tension)

---

**Your machine is now configured for optimal performance with GRBL v1.1f!**

Continue fine-tuning based on actual cutting results and material behavior.
