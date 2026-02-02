# PS2 Controller Integration

### Summary

Successfully integrated a Bluetooth PS2 controller (connected at `/dev/input/js0`) with the unitree_sdk2 framework. The controller data is now packed into the 40-byte `wireless_remote`  field in the  `rt/lowstate`  message.

### Files Created

#### `ps2_joystick.h`

Header file defining:
-   `PS2JoystickState`  structure for storing controller state
-   `WirelessRemoteData`  packed structure (40 bytes) for unitree_sdk2 format
-   Key bitmask constants (`PS2_KEY_A`,  `PS2_KEY_B`,  `PS2_KEY_L1`, etc.)
-   API functions for joystick operations

#### `ps2_joystick.c`
Implementation using Linux joystick API:
-   Non-blocking read from `/dev/input/js0`
-   Maps PS2 axes (lx, ly, rx, ry) from int16 to float (-1.0 to 1.0)
-   Maps buttons to 16-bit bitmask matching unitree format
-   Packs data into 40-byte
    
#### `test_ps2_controller.c`

Standalone test program with visual feedback:
-   ASCII art joystick position display
-   Button state indicators
-   Hex dump of packed  `wireless_remote[40]`  data

### Modified files

-   `src/ec_shared_mem.h`  - Added  `JoystickData`  structure
-   `src/unitree_bridge.cpp`  - Integrated joystick reading and packing into  `wireless_remote[40]`
-   `src/CMakeLists.txt`  - Added build target

### Fixing PS2 Button Mapping

Updated the button mapping in `ps2_joystick.h` to match the controller:

Button | Index
-------|-----
X (Square) |	3
Y (Triangle) |	4
L1 |	6
R1 |	7
L2 |	8 (AXIS 5)
R2 |	9 (AXIS 4)
Select |	10
Start |	11
D-pad |	AXIS 6/7

### Fixing Wireless Remote Format

Fixed the `wireless_remote` data format to match the *unitree* `xRockerBtnDataStruct` structure. The issue was that I had the wrong byte layout.

**Correct format (40 bytes)**:

Bytes |	Field |	Description
------|-------|-------------
0-1 |	head[2] |	Header (zeros)
2-3 |	keys |	Button bitmask (uint16)
4-7 |	lx |	Left stick X (float)
8-11 |	rx |	Right stick X (float)
12-15 |	ry |	Right stick Y (float)
16-19 |	L2 |	L2 trigger (float)
20-23 |	ly |	Left stick Y (float)
24-39 |	idle[16] |	Padding

