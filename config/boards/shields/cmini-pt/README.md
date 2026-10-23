# CMini-PT: two PMW3610 trackballs and a dongle

Both keyboard halves are BLE peripherals. The dongle is the central and runs
the keymap and input listeners. This configuration uses `zmkfirmware/zmk` main,
including Zephyr's PMW3610 driver and ZMK's `zmk,input-split` transport. No
external pointing modules or forked ZMK branches are required.

## Build targets

| Part | Board | Shield |
| --- | --- | --- |
| Left half | `nice_nano@2.0.0//zmk` | `cmini-pt_left` |
| Right half | `nice_nano@2.0.0//zmk` | `cmini-pt_right` |
| Dongle | `nice_nano@2.0.0//zmk` | `cmini-pt_dongle` |

The former `cmini-pt_trackpad` and `cmini-pt_trackball` targets are replaced
by the explicit left/right names above. `build.yaml` and the GitHub Actions
workflow have been updated accordingly. The separate, non-pointing `cmini`
targets remain in the matrix; their board names and pointing key includes are
also migrated for the shared upstream dependency.

## Sensor wiring

Both halves use the original right half's wiring, as confirmed for the new
left sensor. The shared hardware definition is `cmini-pt-pmw3610.dtsi`.

| PMW3610 signal | nRF52840 GPIO | Pro Micro pin |
| --- | --- | --- |
| SCK | P0.06 | D1 |
| SDIO (MOSI and MISO) | P0.20 | D3 |
| CS, active low | P0.17 | D2 |
| MOTION, active low with pull-up | P0.08 | D0 |

Both sensors use 1000 CPI and smart mode. The original swap-X/Y plus invert-X
and invert-Y settings are now input processors on each peripheral's input
split. Adjust these in `cmini-pt.keymap` if the left sensor's physical mounting
requires a different orientation.

## Pointing behavior

| Control | Behavior |
| --- | --- |
| Left ball | Horizontal/vertical scrolling at 1/8 scale |
| Left ball on NUM or SYS | Scrolling at 1/16 scale |
| Right ball | Cursor movement |
| Right ball on NAV | Precision movement: X at 1/4, Y at 1/8 |
| Right ball on MOUSE | Horizontal/vertical scrolling at 1/8 scale |

The left ball keeps the former trackpad's scrolling role, with two-axis
scrolling now available from the ball. Key bindings, combos, and hold-taps are
otherwise preserved. Each layer override replaces the base processor chain;
this avoids applying the normal and precision scalers to the same event.

Split input IDs are 0 for left and 1 for right. Both are defined on all three
parts. Each peripheral enables only its own split and attaches its local
sensor; the dongle enables both splits and both listeners.

The old driver's `CONFIG_PMW3610_*` rest/sample timing settings have no
equivalent configuration in the bundled upstream driver. The sensors use the
upstream driver's power-management defaults instead.

## Build and flash

Push the migration branch to run the upstream reusable GitHub Actions build.
Download the `firmware` artifact after all matrix entries succeed.

For this migration, flash the `settings_reset` image on **both halves and the
nice!nano dongle**, then flash each part's matching new firmware. Remove the
old host Bluetooth pairing and pair again if using the dongle over Bluetooth.
All three parts must run the new firmware together; the old relay protocol is
not compatible with the upstream input-split protocol.

The unrelated SliceMK dongle board has also been migrated to Zephyr hardware
model v2, retaining its original application and storage partition addresses
for its existing UF2 bootloader.

## Validation scope

The migration was checked against upstream ZMK commit
`4dd95d7840eb3d5a3f59bb29fbf1ab054715ef85` and its Zephyr
`v4.1.0+zmk-fixes` dependency. `revision: main` intentionally tracks future
upstream changes, as requested.

All eight build-matrix entries passed devicetree preprocessing and Zephyr
devicetree binding/reference validation. Additional checks verified the split
IDs, sensor references, enabled/disabled listener topology, and 1000 CPI.
These are configuration checks, not firmware compilation or hardware testing.
The full GitHub Actions builds and physical pointing/orientation tests remain
required before flashing for everyday use. Existing deprecated keymap labels
and `global-quick-tap` properties still produce non-fatal warnings.
