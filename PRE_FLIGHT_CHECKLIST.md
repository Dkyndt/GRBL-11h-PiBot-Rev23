# GRBL Upload Pre-Flight Checklist
## PIBOT Controller Rev 2.3 - Safety & Setup Verification

---

## ⚠️ BEFORE CONNECTING POWER

### Hardware Inspection
- [ ] All stepper motor connectors properly seated
- [ ] No loose wires or shorts visible
- [ ] Motor cables not damaged or frayed
- [ ] Limit switch wiring completed and secure
- [ ] Emergency stop button functional
- [ ] Power supply rated for minimum 12A at 12V
- [ ] Power supply voltage verified: 11-12V DC
- [ ] All mounting screws tight
- [ ] Machine mechanically sound (no wobble, binding, or loose parts)

### Switch & Jumper Settings
- [ ] Control board switch set to **"3D"** position
- [ ] External stepper drivers **POWERED OFF** (critical for upload)
- [ ] Stepper driver current properly adjusted (2.66A setting verified)
- [ ] All driver enable pins configured correctly

---

## 💻 SOFTWARE PREPARATION

### Arduino IDE Setup
- [ ] Arduino IDE installed (version 1.8.x or newer)
- [ ] boards.txt modified with PIBOT board definition (administrator rights used)
- [ ] `PiBot_GRBL11-DEF` folder copied to Arduino libraries as `PiBot_CNC_GRBL11`
- [ ] Arduino IDE restarted after library installation

### Configuration Files Verified
- [ ] `config.h` has `DEFAULTS_PIBOT_REV23` enabled (line 41)
- [ ] `config.h` has `CPU_MAP_2560_PIBOT_REV23` enabled (line 42)
- [ ] `config.h` has `BAUD_RATE 115200` set (line 47)
- [ ] `defaults.h` updated with custom steps/mm values:
  - X: 62.75 steps/mm ✓
  - Y: 62.75 steps/mm ✓
  - Z: 400.0 steps/mm ✓
- [ ] `defaults.h` updated with max speeds:
  - X: 3896 mm/min ✓
  - Y: 3896 mm/min ✓
  - Z: 916.8 mm/min ✓
- [ ] `defaults.h` updated with travel limits:
  - X: 770 mm ✓
  - Y: 690 mm ✓
  - Z: 42 mm ✓

---

## 🔌 UPLOAD PROCEDURE

### Step 1: Connect to Computer
- [ ] USB cable connected from computer to PIBOT controller
- [ ] Green power LED on PIBOT board illuminated
- [ ] Device detected in Windows Device Manager / System Profiler
- [ ] COM port number identified: COM_____ (write it down)

### Step 2: Arduino IDE Configuration
- [ ] Arduino IDE opened
- [ ] File → Examples → PiBot_CNC_GRBL11 → PiBotUploader selected
- [ ] Tools → Board → **"PiBot Controller Rev2.x"** selected
- [ ] Tools → Port → Correct COM port selected
- [ ] Tools → Programmer → **"AVRISP mkII"** or **"Arduino as ISP"** selected

### Step 3: Verify Code
- [ ] Click ✓ **Verify/Compile** button
- [ ] Compilation successful (no errors)
- [ ] Sketch size shown (should be ~45-50KB of 253,952 bytes)
- [ ] RAM usage shown (should be ~2-3KB of 8,192 bytes)

### Step 4: Upload Firmware
- [ ] **FINAL CHECK**: External stepper drivers are **OFF**
- [ ] Click → **Upload** button
- [ ] Wait for "Compiling sketch..." message
- [ ] Wait for "Uploading..." message
- [ ] Progress bar reaches 100%
- [ ] "Done uploading" message appears
- [ ] No error messages in console window

### Expected Upload Time
Upload takes approximately **15-25 seconds**. Do not disconnect during upload!

---

## ✅ POST-UPLOAD VERIFICATION

### Step 1: Initial Communication
- [ ] Open Arduino IDE Serial Monitor (Tools → Serial Monitor)
- [ ] Set baud rate to **115200**
- [ ] Set line ending to **"Newline"** or **"Both NL & CR"**
- [ ] Type `$I` and press Enter
- [ ] Verify response contains "1.1h" (e.g., `[VER:1.1h:<build>:...]`) — upstream commit bfb67f0
- [ ] Type `$$` and press Enter
- [ ] Settings list appears (32 lines starting with $0=...)

### Step 2: Settings Verification
- [ ] `$100=62.750` (X steps/mm) ✓
- [ ] `$101=62.750` (Y steps/mm) ✓
- [ ] `$102=400.000` (Z steps/mm) ✓
- [ ] `$110=3896.000` (X max rate) ✓
- [ ] `$111=3896.000` (Y max rate) ✓
- [ ] `$112=916.800` (Z max rate) ✓
- [ ] `$130=770.000` (X max travel) ✓
- [ ] `$131=690.000` (Y max travel) ✓
- [ ] `$132=42.000` (Z max travel) ✓
- [ ] `$22=1` (Homing enabled) ✓

### Step 3: Power Up Stepper Drivers
- [ ] Disconnect USB (power off PIBOT)
- [ ] **NOW** turn ON external stepper driver power supply
- [ ] Verify stepper driver power LEDs illuminated
- [ ] Verify no smoke, burning smell, or unusual sounds
- [ ] Reconnect USB to PIBOT
- [ ] Verify GRBL still responds to `?` command

---

## 🔧 INITIAL TESTING (NO LOAD)

### Motor Test (Drivers ON, Machine NOT Moving Yet)
- [ ] Send `$X` to unlock machine
- [ ] Machine responds with `ok`
- [ ] No error messages
- [ ] Motors holding position (feel slight resistance when trying to turn by hand)

### Jog Test (Low Speed)
- [ ] Send: `G91` (relative mode)
- [ ] Send: `G21` (mm mode)
- [ ] Send: `F100` (slow feed rate)
- [ ] Send: `G0 X10` (move X 10mm)
- [ ] **WATCH CAREFULLY** - X motor should move smoothly
- [ ] Stop immediately if motor stalls or makes grinding noise
- [ ] Send: `G0 X-10` (return X)
- [ ] Repeat for Y and Z axes

### Direction Verification
- [ ] X positive moves **RIGHT** (if not, note for adjustment)
- [ ] Y positive moves **BACK/AWAY** (if not, note for adjustment)
- [ ] Z positive moves **UP** (if not, note for adjustment)

---

## ❌ TROUBLESHOOTING

### Upload Fails - "avrdude: stk500_recv(): programmer is not responding"
**Solution:**
- Check USB cable connection
- Try different USB port
- Verify correct board and port selected
- Ensure no other software using COM port
- Try pressing reset button on PIBOT just before upload starts

### Upload Fails - "espcomm_upload_mem failed"
**Solution:**
- Wrong board selected (should be "PiBot Controller Rev2.x")
- Check boards.txt was modified correctly

### No Response After Upload
**Solution:**
- Check baud rate is 115200
- Check line ending is set (Newline or Both NL & CR)
- Press reset button on PIBOT
- Reconnect USB cable

### Motor Stalls/Skips Steps
**Solution:**
- Check stepper driver current adjustment (should be set for 2.66A)
- Verify wiring to motors correct
- Reduce speed: `$110=2000`, `$111=2000`, `$112=500`
- Reduce acceleration: `$120=360000`, `$121=360000`, `$122=180000`

### Wrong Direction of Movement
**Solution:**
- Adjust `$3` setting (direction invert mask)
- Swap motor wiring pairs at driver or motor
- See QUICK_REFERENCE.md for detailed instructions

### Limit Switches Not Working
**Solution:**
- Check wiring connections
- Test continuity with multimeter
- Try inverting: `$5=1`
- Disable temporarily for testing: `$21=0`

---

## 📊 EXPECTED PERFORMANCE

### Normal Operation Indicators
✅ Motors run smoothly without noise
✅ No missed steps during movements
✅ Consistent positioning after multiple moves
✅ Motors not overheating (warm is OK, too hot to touch is NOT)
✅ Power supply voltage stays stable under load (>10.5V)
✅ No burning smell from drivers or motors

### Warning Signs
⚠️ High-pitched whining from motors → Reduce speed/acceleration
⚠️ Grinding/clicking sounds → Mechanical binding or incorrect direction
⚠️ Motors very hot (>60°C) → Reduce driver current or check wiring
⚠️ Position drift after moves → Missed steps, reduce speed
⚠️ Stuttering motion → Increase step pulse time: `$0=20`

---

## 🎯 FINAL VERIFICATION

Before cutting operations:
- [ ] All axes move correctly in both directions
- [ ] Homing cycle completes successfully (`$H` command)
- [ ] Soft limits prevent overtravel
- [ ] Emergency stop tested and functional
- [ ] Measured test square matches commanded dimensions (±0.5mm)
- [ ] Machine coordinate system properly calibrated
- [ ] Work coordinate system (G54) set correctly
- [ ] All safety features enabled and tested
- [ ] `CONFIGURATION_SUMMARY.md` reviewed
- [ ] `QUICK_REFERENCE.md` printed and available

---

## 📝 NOTES SECTION

### Upload Date: _____________________

### COM Port Used: _____________________

### Compilation Results:
- Sketch size: _________ KB / 253,952 bytes (___%)
- Global variables: _________ KB / 8,192 bytes (___%)

### Initial Test Results:
- X-axis direction: ☐ Correct  ☐ Inverted
- Y-axis direction: ☐ Correct  ☐ Inverted  
- Z-axis direction: ☐ Correct  ☐ Inverted

### Settings Adjustments Made:
```
(Record any $ commands used to modify settings)




```

### Issues Encountered:
```
(Note any problems and solutions)




```

### Final Status:
☐ **READY FOR OPERATION** - All tests passed
☐ **NEEDS CALIBRATION** - Upload successful, needs fine-tuning
☐ **ISSUES REMAIN** - See notes above

---

## 📞 SUPPORT RESOURCES

- GRBL Wiki: https://github.com/gnea/grbl/wiki
- GRBL Configuration: https://github.com/gnea/grbl/wiki/Grbl-v1.1-Configuration
- PIBOT Support: https://www.pibot.com
- G-Code Reference: https://linuxcnc.org/docs/html/gcode.html

**Configuration completed by:** _____________________
**Date:** _____________________
**Verified by:** _____________________

---

**STATUS**: ✅ **Configuration is ready for upload!**

All custom settings have been applied to your GRBL firmware:
- Steps/mm calibrated for your belt and screw drives
- Speed limits matched to motor specifications  
- Travel limits set for your machine size
- Safety features enabled
- Homing configured

**You can now proceed with the Arduino upload process.**
