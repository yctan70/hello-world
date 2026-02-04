## Reconfigure Motor Types

### Changes to `g1_robot_config.h`
- Master 0 count: Changed from 9 to 6 (Left Leg only)
- Master 2 count: Changed from 7 to 10 (Left Arm + Waist)
- Waist motors (joints 12-14): Changed from SE motors on Master 0 (slaves 6-8) to LS motors on Master 2 (slaves 7-9)

### Changes to `ec_rt_thread.c`
- Updated master initialization to configure Master 0 with 6 slaves instead of 9
- Updated Master 2 to configure 10 slaves (positions 0-9) instead of 7
- Updated comments to reflect the new configuration

### New Motor Layout
| Master  | Motor Type | Motors |
|----------|---------------|------|
| 0 |	SE |	Left Leg (6 motors) |
| 1 | SE |	Right Leg (6 motors) |
| 2 | LS |	Left Arm (7) + Waist (3) = 10 motors |
| 3 |	LS |	Right Arm (7 motors) |

The waist motors will now use the CiA 402 CST (Cyclic Synchronous Torque) mode with impedance control wrapper, matching the arm motors.

## LS Hand Integration

Successfully integrated left and right LS dexterous hands into the G1 robot EtherCAT control system.

### Changes Made

#### New Files

| File | Description |
|------|-------------|
| `src/ls_hand.h` | Header with `LSHandContext` struct and public API |
| `src/ls_hand.c` | Implementation using `LHandProLib` for `PDO` handling |

#### Modified Files

#### `src/constants.h`
- Added `G1_NUM_HANDS` (2), `HAND_DOF` (6), `HAND_SENSOR_COUNT` (12)
- Added `HAND_LEFT`/`HAND_RIGHT` indices

#### `src/ec_shared_mem.h`
- Added `HandData` struct with motor positions, sensor data, and control state
- Added `hands[G1_NUM_HANDS]` array to `SharedMemoryData`

#### `src/ec_rt_thread.c`
- Added hand context variables and `ls_hand.h` include
- Added `ls_hand_init()` calls after motor configuration
- Added `ls_hand_process()` calls in RT cyclic loop
- Added `ls_hand_close()` calls in cleanup

#### `src/CMakeLists.txt`
- Added LHandProLib include and library paths
- Added `ls_hand.c` to ec_rt_thread sources
- Added RPATH for runtime library loading

### Configuration
| Hand | Master | Slave Position |
|------|--------|----------------|
| Left | 2 | 10 |
| Right | 3 | 7 |

### Build
```bash
cd /home/yctan/g1_motor_control/build
cmake -DUSE_MOCK_ECAT=ON .. # For simulation
cmake .. # For real hardware
make ec_rt_thread
```

### Run
```bash
sudo ./bin/ec_rt_thread
```

## LS Hand DDS Integration

### Summary
Integrated DDS communication for LS hands into the G1 robot control system, enabling hand control via Unitree SDK2-compatible DDS topics.

## Changes Made

### IDL Structures ( `src/unitree_hg.idl`)
Added the following DDS message types:
| Structure | Description |
|-----------|-------------|
| `PressSensorState_` | 12-sensor pressure/temperature data |
| `HandMotorCmd_` | Command for a single hand motor |
| `HandMotorState_` | State of a single hand motor |
| `HandCmd_` | Complete command for 6-DOF hand |
| `HandState_` | Complete state with sensors |

### Generated Files
Regenerated C bindings via `idlc -l c unitree_hg.idl`:
- `src/unitree_hg.h`
- `src/unitree_hg.c`


### Bridge Updates (`src/unitree_bridge.cpp`)
- Added DDS topics: `rt/hand/left/cmd`, `rt/hand/left/state`, `rt/hand/right/cmd`, `rt/hand/right/state`
- Implemented `initialize_hand_topics()` for DDS entity creation
- Implemented `publish_hand_state()` to publish hand states from shared memory
- Implemented `process_hand_cmd()` to receive hand commands into shared memory

## Testing
Build verified:
```bash
cd build && cmake -DUSE_MOCK_ECAT=ON .. && make
```
Result: **SUCCESS** (minor unused parameter warnings only)

## Next Steps
1. Test with `ec_rt_thread` running to verify hand data flows through DDS
2. Create a Python subscriber to verify hand state topics
3. Test command reception from external DDS publisher


## Create A Test Program
Created `…/g1_motor_control/src/g1_hand_example.cpp` - a test application for the LS dexterous hand, modeled after Unitree's `g1_dex3_example.cpp` but adapted for your project:

### Key differences from the original Unitree example:
1. **Uses CycloneDDS directly** instead of *ROS2* (`rclcpp`)
2. **Uses your project's IDL structures** (unitree_hg_msg_dds__HandCmd_, unitree_hg_msg_dds__HandState_)
3. **Connects to your unitree_bridge topics**: `rt/hand/left/cmd`, `rt/hand/left/state`, `rt/hand/right/cmd`, `rt/hand/right/state`.
4. **Simplified 6-DOF control** (vs. *Unitree*'s 7-DOF `Dex3` hand)

### Features:
- **Interactive keyboard control** with the same commands as the original:
	- **r** - Rotate (sinusoidal motion through range)
	- **g** - Grip (close to middle position)
	- **s** - Stop motors
	- **p** - Print current state
	- **h** - Help
	- **q** - Quit
- **Multi-threaded design**: Separate threads for control loop, user input, and state reading
- **State display**: Shows motor positions, velocities, currents, sensor data (normal force), and error codes

