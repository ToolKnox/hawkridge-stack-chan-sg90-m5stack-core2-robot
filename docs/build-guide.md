# Build guide — Stack-chan SG90 Desktop Robot for M5Stack Core2

Skill level: comfortable with a 3D printer, small screws and a terminal. No soldering.
Time: a few hours of printing and about an hour of assembly, plus the first firmware setup.

## 1. Print the parts

| File | Qty | Supports | Notes |
|---|---|---|---|
| `shell_SG90_gobottom_v1.0.stl` | 1 | yes | lay it with the M5Stack opening facing down |
| `feet_SG90_v1.0.stl` | 1 | no | carries the pan servo |
| `bracket_SG90_v1.1.stl` | 1 | no | pan/tilt bracket for TowerPro SG90-size servos |
| `bracket_MG90S_v1.0.stl` | 0 or 1 | no | use instead of the SG90 bracket for genuine MG90S servos |
| `spacer_v1.0.stl` | 1 | no | used when you build without taking the Core2 apart |
| `hat_base_v1.0.stl`, `hat_cat_ear_v2.0.stl` | optional | no | hats that sit on the head |

PLA at 0.2 mm layers. STEP files for the shell, feet and SG90 bracket are included so you can change the design.

## 2. Parts to buy

- M5Stack Core2 with the M5GO Bottom2 battery module. The Core2 for AWS IoT kit already includes it.
- M5Stack Servo Kit 180: two SG90-size servos and two Grove to Servo adapters.
- 8 M2 x 5 mm screws for the bracket, feet and shell.
- 2 M3 x 18 mm and 2 M3 x 15 mm screws for the bottom module. The Core2 for AWS kit ships with these.
- Small zip ties, a USB-C data cable (charge-only cables will not flash), 1.5 mm and 2.5 mm hex drivers and a
  small Phillips driver.

The servo kit adapters connect each servo to its own Grove port, so you do not need a branch cable.

## 3. Set up the firmware tools

On a Linux, macOS or Windows PC with git and Node.js 22:

1. `git clone https://github.com/stack-chan/stack-chan.git`, then `cd stack-chan/firmware` and `npm i`.
2. `npm run setup`, then `npm run setup -- --device=esp32`. This installs the Moddable SDK and ESP-IDF.
3. `npm run doctor` should list esp32 as a supported target.

The firmware is also mirrored in this project's repository under `source/upstream/firmware/`.

## 4. Choose the servo ports and pins

The Core2 target (`takao_core2_sg90`) defaults to pan GPIO 19 and tilt GPIO 27, which belong to a dedicated base
board. With the M5GO Bottom2 you use two Grove ports instead:

| Port | GPIO pins | Use | Notes |
|---|---|---|---|
| Port A (red) | 33, 32 | pan servo | |
| Port C (blue) | 13, 14 | tilt servo | |
| Port B (black) | 26, 36 | avoid | GPIO 36 is input-only |

Each Grove to Servo adapter passes one signal line of its port to the servo. On a standard Grove cable that is usually
the yellow wire: GPIO 33 on Port A and GPIO 13 on Port C. Put the driver settings in
`firmware/host/app/manifest_local.json`:

```json
{
  "config": {
    "driver": { "type": "pwm", "pwmPan": 33, "pwmTilt": 13 }
  }
}
```

If a servo stays still after flashing, set that axis to the other pin of the same port (32 or 14) and flash again.

## 5. Flash and test before assembly

1. Plug the servos into the adapters and the adapters into Port A and Port C.
2. Connect the Core2 over USB-C and run `npm run flash:takao_core2_sg90` in `stack-chan/firmware`.
3. The face appears and the head servos sweep. Let each servo settle at its center (90 degrees), then fit the
   horns so the face will point straight ahead.

## 6. Assembly

1. Snap the pan servo into the feet and the tilt servo into the bracket. Use the small screws that come with the
   servos for the horns.
2. Join the feet, bracket and shell with the M2 x 5 mm screws.
3. Route the servo cables through the bracket and tidy them with zip ties.
4. Fit the Core2 into the shell and fix the bottom module with the M3 screws (1.5 mm and 2.5 mm hex drivers).
5. Add a hat if you like.

## 7. Use it

Use the Stack-chan browser tools to change Wi-Fi and preferences, try MODs, or make your own face. Install a MOD
with `npm run mod:takao_core2_sg90 -- <path to mod manifest>`.

## Troubleshooting

- The Core2 resets when the servos start: SG90 servos draw a short burst of current. Charge the battery fully.
- A servo jitters or does not move: check the port and the pin number in `manifest_local.json`.
- The bracket does not fit: some servos sold as SG90 or MG90S differ in size or horn shape. Check yours against
  the bracket before you print many copies. Genuine MG90S servos may also need more current than the battery gives.
- Flashing fails: use a USB-C data cable and check that `npm run doctor` lists esp32.

## Safety

Keep fingers clear of the neck joint while the servos move. Do not leave the Li-ion battery charging unattended.
