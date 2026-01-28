# Cornelius ZMK Configuration Guide

## File Overview

This ZMK keyboard firmware configuration consists of several key files:

### 1. **cornelius.dtsi** - Device Tree Source Include (Hardware Definition)
**Purpose**: Defines the physical hardware layout and GPIO pin assignments.

**Key Components**:
- **Matrix Transform**: Maps the physical 3x7 key layout to logical key positions
  - 4 rows × 7 columns per side = 28 keys per half (56 total)
  - Row 0 (top): 6 keys per side (no aux)
  - Row 1 (home): 7 keys per side (includes aux column)
  - Row 2 (bottom): 7 keys per side (includes aux column)
  - Row 3 (thumb): 3 keys per side
  
- **KSCAN Configuration**: Keyboard Scan defines how the matrix is read
  - **Row Pins**: P009, P010, P111, P113 (outputs)
  - **Column Pins**: P106, P104, P011, P100, P024, P022, P020 (inputs)
  - **Diode Direction**: `col2row` - prevents ghosting in multi-key presses
  - **Debouncing**: 1ms press, 10ms release (adjust if keys feel bouncy)

### 2. **cornelius_left.overlay** & **cornelius_right.overlay** - Side-Specific Configs
**Purpose**: Apply board-specific settings for left and right halves.

**Why separate files?**
- On typical split keyboards, left and right sides have different pin assignments
- Your design uses the same pins on both sides, so these files are nearly identical
- The overlay system allows flexibility if you later add side-specific features (OLED, encoders, etc.)

**Current Content**:
- Includes the base `cornelius.dtsi` configuration
- Disables I2C (not needed without OLED/advanced features)

### 3. **cornelius.keymap** - Layer Definitions
**Purpose**: Defines what each key does on each layer.

**Layers**:
- **Layer 0 (Base/default_layer)**: QWERTY layout
  - Uses `&mo 1` to momentarily switch to lower layer (hold the key)
  - Uses `&mo 2` to momentarily switch to raise layer
  - Aux column has `&trans` (transparent) - passes through to lower layer
  
- **Layer 1 (lower_layer)**: Numbers and Bluetooth
  - Numbers 1-5 on top row
  - Bluetooth clear and device selection on second row
  - Arrow keys (LEFT, DOWN, UP, RIGHT)
  
- **Layer 2 (raise_layer)**: Symbols and Function
  - Special characters (!@#$%^&*()etc.)
  - Math operators (-=+_)
  - Brackets and braces
  - Backtick and tilde

**Key Binding Examples**:
- `&kp ESC` - Key press: Sends ESC character
- `&mo 1` - Momentary layer: Hold to activate layer 1, release to return
- `&trans` - Transparent: Pass the key through to the layer below
- `&bt BT_SEL 0` - Bluetooth: Select device 0

### 4. **cornelius.conf** - Configuration Settings
**Purpose**: Enables/disables ZMK features via Kconfig.

**Current Settings**:
- `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` - Increases Bluetooth range (uses more power)
- `CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=1` - 1ms debounce on key down
- `CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=10` - 10ms debounce on key up
- `CONFIG_ZMK_STUDIO=y` - Enables ZMK Studio for live keymap editing

**Optional Settings** (uncomment to enable):
- `CONFIG_ZMK_SLEEP=y` - Deep sleep to extend battery life
- `CONFIG_ZMK_USB_LOGGING=y` - USB logging (helps with debugging)

### 5. **west.yml** - Dependency Management
**Purpose**: Manages ZMK firmware versions and dependencies.

**Current Config**:
- `revision: v0.3` - Uses ZMK version 0.3 (stable)
- This file typically doesn't need changes unless you need specific ZMK features

## Building and Flashing

### Build Steps:
1. Ensure you have `west` installed
2. From the workspace root: `west build -b nice_nano_v2 -d build/left -- -DSHIELD=cornelius_left`
3. For right side: `west build -b nice_nano_v2 -d build/right -- -DSHIELD=cornelius_right`

### Flashing:
1. Enter bootloader mode on the nice!nano (double-tap reset button)
2. Copy the `.uf2` file from `build/left/zephyr/` or `build/right/zephyr/`
3. Paste into the mounted USB drive (appears as `NICENANO` or `NRF52BOOT`)

## Customizing Your Keymap

### Adding Keys to Aux Column:
In `cornelius.keymap`, replace `&trans` with desired keys:

```
// Example: Add volume control to aux column
&kp C_VOLUME_UP      // Volume up on home row aux
&kp C_VOLUME_DOWN    // Volume down on bottom row aux
&kp C_MUTE           // Mute on thumb aux
```

### Common Key Codes:
- Letters/Numbers: `&kp A`, `&kp N1`, etc.
- Modifiers: `&kp LCTRL`, `&kp LALT`, `&kp LSHFT`
- Media: `&kp C_VOLUME_UP`, `&kp C_PLAY_PAUSE`
- Function Keys: `&kp F1` through `&kp F12`
- Special: `&kp BSPC` (backspace), `&kp SPC` (space), `&kp RET` (return)

### Advanced Behaviors:
- Layer toggle: `&tog 1` (press to switch, press again to switch back)
- Tap-hold: Requires defining a behavior in `.dtsi`
- Mod-tap: Quick tap = one behavior, hold = modifier

## Troubleshooting

### Keys Not Responding?
1. Check `cornelius.dtsi` - verify GPIO pins match your wiring
2. Check matrix orientation (rows vs columns)
3. Enable `CONFIG_ZMK_USB_LOGGING=y` in `.conf` to debug

### Split Keyboard Not Syncing?
1. Ensure both halves are built and flashed
2. Check ZMK Studio status LED or logs
3. May need to clear Bluetooth bonds: Hold Bluetooth clear key

### Battery Draining Too Fast?
1. Uncomment `CONFIG_ZMK_SLEEP=y` in `.conf`
2. Reduce `CONFIG_BT_CTLR_TX_PWR_PLUS_8` if Bluetooth range isn't critical
3. Disable `CONFIG_ZMK_STUDIO=y` when not actively editing

## Resources

- ZMK Documentation: https://zmk.dev
- ZMK Discord: https://discord.gg/8gkDtKBA3D
- Nice!nano Guide: https://docs.nicekeyboards.com/nice!nano/
- Device Tree Documentation: https://docs.zephyrproject.org/latest/build/dts/

## Next Steps

1. **Build and Flash**: Test both halves with the default keymap
2. **Customize Aux Column**: Add your preferred keys to the 7th column
3. **Adjust Debouncing**: If keys feel bouncy, increase debounce values
4. **Enable Deep Sleep**: Add `CONFIG_ZMK_SLEEP=y` to extend battery life
5. **Add Features**: Consider encoders, OLED displays, RGB LEDs (supported by ZMK)
