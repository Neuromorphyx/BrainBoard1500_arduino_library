# BB15 Arduino Library

`BB15` is the board-centric Arduino library for BrainBoard1500.

The public API is centered on the physical BB15 board:

- `BB15` handles board bring-up, reset routing, expander access, flash access,
  and transport state.
- `BB15Model` describes a serialized Akida model and where it lives.
- `BB15Runner` loads models and runs inference.

The constructor requires a `BB15Pinout`, so host-board wiring is always
explicit:

```cpp
#include <BB15.h>

BB15Pinout pinout = BB15Pinout::niclaSenseMeDefaults();
BB15Config config = BB15Config::defaults();

void setup() {
  static BB15 bb15(pinout, config);
}
```

Construct `BB15` once `setup()` is running. Do not create it as a global
object, because the constructor touches board hardware state.

## Supported Hosts

The current target hosts are:

- Arduino Nicla Sense ME
- Arduino Nicla Vision

Classic AVR boards are not a target for this library.

## Included Examples

The example set is intentionally small and board-specific:

1. `examples/bb15_model_flasher_nicla_sense`
   Flashes one exported Akida model into BB15 external flash from a Nicla Sense
   ME host and verifies that the runtime can load it back.
2. `examples/bb15_dummy_inference_nicla_sense`
   Loads the flashed model and runs interrupt-driven synthetic inference on a
   Nicla Sense ME host.
3. `examples/bb15_model_flasher_nicla_vision`
   Flashes one exported Akida model into BB15 external flash from a Nicla
   Vision host and verifies that the runtime can load it back.
4. `examples/bb15_dummy_inference_nicla_vision`
   Loads the flashed model and runs interrupt-driven synthetic inference on a
   Nicla Vision host.
5. `examples/bb15_nicla_vision_human_detection`
   Captures live Nicla Vision camera frames, preprocesses them for the bundled
   VWW person-detection model, runs interrupt-completed inference, and streams
   a grayscale preview plus results over USB CDC.

All sketches are heavily commented and meant to be modified by users.

## Getting Started

1. Export your model into the example asset files:
   - `program.h`
   - `program.cpp`
   - `model_metadata.h`
   - `model_metadata.cpp`
2. Pick the example pair that matches your host board.
3. Upload the matching `bb15_model_flasher_*` sketch.
4. Confirm the sketch reports a successful flash and verify pass.
5. Upload the matching `bb15_dummy_inference_*` sketch.
6. Confirm inference completes and prints scores.

### Nicla Vision Human Detection

The Vision flasher installs
`NiclaV_VWW_PersonDet_EN_USBbottom_2026-06-14`, a `96x96x3` Visual Wake Words
person-detection model. The matching live-camera workflow is:

```bash
arduino-cli compile --clean --fqbn arduino:mbed_nicla:nicla_vision --library . examples/bb15_model_flasher_nicla_vision
arduino-cli upload --fqbn arduino:mbed_nicla:nicla_vision --port /dev/ttyACM0 examples/bb15_model_flasher_nicla_vision
arduino-cli compile --clean --fqbn arduino:mbed_nicla:nicla_vision --library . examples/bb15_nicla_vision_human_detection
arduino-cli upload --fqbn arduino:mbed_nicla:nicla_vision --port /dev/ttyACM0 examples/bb15_nicla_vision_human_detection
python3 -m pip install -r tools/requirements.txt
python3 tools/bb15_nicla_vision_preview.py --port /dev/ttyACM0
```

Upload the flasher first, confirm flash/verify/model-load success, then upload
the human-detection sketch. Replace `/dev/ttyACM0` with the serial port assigned
by your operating system. The camera sketch never writes flash. It captures
RGB565 at `320x240`, sends a grayscale USB preview, and converts each frame to
the VWW model's rotated `96x96x3` input locally. The desktop tool requires
Python 3.7 or later, `pyserial`, and the system Tk package used by `tkinter`.

## Arduino CLI Build Checks

The current validated compile commands are:

```bash
arduino-cli compile --clean --fqbn arduino:mbed_nicla:nicla_sense --library . examples/bb15_model_flasher_nicla_sense
arduino-cli compile --clean --fqbn arduino:mbed_nicla:nicla_sense --library . examples/bb15_dummy_inference_nicla_sense
arduino-cli compile --clean --fqbn arduino:mbed_nicla:nicla_vision --library . examples/bb15_model_flasher_nicla_vision
arduino-cli compile --clean --fqbn arduino:mbed_nicla:nicla_vision --library . examples/bb15_dummy_inference_nicla_vision
arduino-cli compile --clean --fqbn arduino:mbed_nicla:nicla_vision --library . examples/bb15_nicla_vision_human_detection
```

Run these sequentially. Parallel `arduino-cli` compiles can race in the shared
Arduino cache and produce misleading failures.

## Validation Status

Validation status as of August 21, 2026:

- the four existing primary examples compile successfully in this repository
- the Nicla Sense ME flasher and dummy inference examples compile for
  `arduino:mbed_nicla:nicla_sense`
- the Nicla Vision flasher and dummy inference examples compile for
  `arduino:mbed_nicla:nicla_vision`
- the Nicla Vision flasher has been hardware-validated in this workspace and
  now completes flash, verify, and model load successfully
- the human-detection camera path has been hardware-validated on Nicla Vision:
  VWW model load, camera capture, preprocessing, interrupt-completed inference,
  and a complete `320x240` USB frame-result packet all succeeded

## Repository Layout

- `src/`
  Public library code and internal runtime support owned by this package.
- `examples/`
  The supported user-facing sketches.
