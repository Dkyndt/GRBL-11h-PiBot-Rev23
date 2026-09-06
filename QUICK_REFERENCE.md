# GRBL Quick Reference - Post Upload Calibration

## Connect to Your PIBOT Controller

### Serial Connection
- **Baud Rate**: 115200
- **Data Bits**: 8
- **Stop Bits**: 1
- **Parity**: None
- **Flow Control**: None

### Recommended Software
- **Universal G-Code Sender (UGS)**: https://winder.github.io/ugs_website/
- **bCNC**: https://github.com/vlachoudis/bCNC
- **Serial Monitor**: Arduino IDE or PuTTY

---

## First Boot Commands

### 1. Check GRBL Version
```
$I
```
Expected response: contains "1.1h" (e.g., `[VER:1.1h:<build>:...]`) — confirm the exact build on your device; README notes upstream commit bfb67f0.

### 2. View All Settings
```
$$
```

### 3. Reset to Defaults (if needed)
```
$RST=$
```

---

## Expected Settings After Upload

```
$0=10    (Step pulse time, µs)
$1=255   (Step idle delay, ms - always enabled)
$2=0     (Step pulse invert, mask)
$3=1     (Step direction invert, mask)
$4=1     (Invert step enable pin, bool)
$5=0     (Invert limit pins, bool)
$6=0     (Invert probe pin, bool)
$10=1    (Status report options, mask)
$11=0.020 (Junction deviation, mm)
$12=0.002 (Arc tolerance, mm)
$13=0    (Report in mm)
$20=1    (Soft limits enable, bool)
$21=1    (Hard limits enable, bool)
$22=1    (Homing cycle enable, bool)
$23=0    (Homing direction invert, mask)
$24=50.000 (Homing locate feed rate, mm/min)
$25=800.000 (Homing search seek rate, mm/min)
$26=250  (Homing switch debounce delay, ms)
$27=2.000 (Homing switch pull-off distance, mm)
$30=1000 (Maximum spindle speed, RPM)
$31=0    (Minimum spindle speed, RPM)
$32=0    (Laser-mode enable, bool)
$100=62.750 (X-axis travel resolution, step/mm)
$101=62.750 (Y-axis travel resolution, step/mm)
$102=400.000 (Z-axis travel resolution, step/mm)
$110=3896.000 (X-axis maximum rate, mm/min)
$111=3896.000 (Y-axis maximum rate, mm/min)
$112=916.800 (Z-axis maximum rate, mm/min)
$120=540000.000 (X-axis acceleration, mm/sec^2)
$121=540000.000 (Y-axis acceleration, mm/sec^2)
$122=180000.000 (Z-axis acceleration, mm/sec^2)
$130=770.000 (X-axis maximum travel, mm)
$131=690.000 (Y-axis maximum travel, mm)
$132=42.000 (Z-axis maximum travel, mm)
```

---

## Testing Procedure

### Step 1: Unlock the Machine
After power-up, machine is locked. Unlock with:
```
$X
```
Or perform homing cycle:
```
$H
```

### Step 2: Jog Each Axis Individually

**Test X-Axis** (10mm at slow speed):
```
G91         (Relative positioning)
G21         (Millimeter mode)
F100        (Feed rate 100 mm/min)
G0 X10      (Move X positive 10mm)
G0 X-10     (Move X negative 10mm)
```

**Test Y-Axis**:
```
G0 Y10      (Move Y positive 10mm)
G0 Y-10     (Move Y negative 10mm)
```

**Test Z-Axis**:
```
G0 Z10      (Move Z positive 10mm)
G0 Z-10     (Move Z negative 10mm)
```

### Step 3: Verify Direction
- **X-Axis**: Positive should move RIGHT
- **Y-Axis**: Positive should move AWAY from front
- **Z-Axis**: Positive should move UP

**If direction is wrong**, invert that axis:
```
$3=X        (Where X is the new invert mask)
```
- To invert X only: `$3=1`
- To invert Y only: `$3=2`
- To invert Z only: `$3=4` (default)
- To invert X+Y: `$3=3`
- To invert X+Z: `$3=5`
- To invert Y+Z: `$3=6`
- To invert all: `$3=7`

### Step 4: Test Limit Switches
Manually trigger each limit switch and verify status changes:
```
?           (Status report - shows limit switch states)
```

### Step 5: Calibrate Steps/mm

**Measure actual movement**:
1. Mark starting position
2. Command move: `G91 G0 X100 F500` (move 100mm)
3. Measure actual distance traveled
4. Calculate: `New_steps = Current_steps × (Commanded / Actual)`

**Example**: If commanded 100mm but actually moved 98mm:
- New X steps = 62.75 × (100 / 98) = 64.03 steps/mm
- Update: `$100=64.03`

Repeat for Y and Z axes.

### Step 6: Test Maximum Speeds
Gradually increase feed rate to find reliable maximum:
```
G91 G0 X100 F500    (Start slow)
G91 G0 X100 F1000   (Increase)
G91 G0 X100 F2000   (Increase)
G91 G0 X100 F3000   (Increase)
```

If motors skip steps, reduce max rate settings:
```
$110=3000    (Reduce X max rate if needed)
```

---

## Common Issues & Solutions

### Issue: Machine locked with "ALARM:1"
**Solution**: Perform homing cycle `$H` or unlock with `$X`

### Issue: Motor moves wrong direction
**Solution**: Adjust `$3` direction invert mask (see Step 3 above)

### Issue: Motors skip steps at high speed
**Solution**: 
- Reduce max rate: `$110`, `$111`, `$112`
- Reduce acceleration: `$120`, `$121`, `$122`
- Check motor current settings on drivers
- Verify power supply voltage

### Issue: Homing fails
**Solution**:
- Check limit switch wiring
- Verify limit switch polarity: `$5=0` (normally) or `$5=1` (if inverted)
- Adjust homing direction: `$23` mask
- Slow down homing: `$24=25` and `$25=400`

### Issue: Machine doesn't move far enough
**Solution**: Increase travel limits `$130`, `$131`, `$132`

### Issue: Soft limit error
**Solution**: 
- Disable temporarily: `$20=0`
- Or home machine first: `$H`

---

## Fine-Tuning Acceleration

Start conservative, then increase if motors can handle it:

**Current Settings**:
- X/Y: 540,000 mm/min² (150 mm/sec²)
- Z: 180,000 mm/min² (50 mm/sec²)

**If you can increase**:
```
$120=720000    (X: 200 mm/sec²)
$121=720000    (Y: 200 mm/sec²)
$122=360000    (Z: 100 mm/sec²)
```

**If you need to decrease** (losing steps):
```
$120=360000    (X: 100 mm/sec²)
$121=360000    (Y: 100 mm/sec²)
$122=180000    (Z: 50 mm/sec²)
```

---

## Saving Modified Settings

Settings are automatically saved to EEPROM after each `$X=` command.

To backup all settings, save the output of `$$` command to a text file.

---

## Emergency Stop

- **Software**: Send `!` (feed hold) then `Ctrl+X` (reset)
- **Hardware**: Use emergency stop button or cut power

---

## Useful G-Code Commands

```
$H          - Home all axes
$X          - Unlock/kill alarm
$$          - View settings
$I          - View version info
$N          - View startup blocks
$C          - Check G-code mode
?           - Status report
!           - Feed hold
~           - Resume
Ctrl+X      - Reset (0x18)
G28         - Go to predefined position
G30         - Go to predefined position 2
```

---

## Performance Monitoring

Monitor for these issues during test cuts:
1. ❌ Missed steps (motor stalling)
2. ❌ Excessive vibration
3. ❌ Overheating motors
4. ❌ Position drift after moves
5. ✅ Smooth acceleration/deceleration
6. ✅ Accurate positioning
7. ✅ Quiet operation

---

## Contact & Support

GRBL Documentation: https://github.com/gnea/grbl/wiki
PIBOT Support: https://www.pibot.com

**Good luck with your CNC setup!** 🛠️
