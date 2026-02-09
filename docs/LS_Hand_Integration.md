# LS Hand Integration Walkthrough

Successfully integrated left and right LS dexterous hands into the G1 robot EtherCAT control system.

## Changes Made

### New Files
| File | Description |
|------|-------------|
| `src/ls_hand.h` | Header with `LSHandContext` struct and public API |
| `src/ls_hand.c` | Implementation using LHandProLib for PDO handling |

### Modified Files

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

## Configuration

| Hand | Master | Slave Position |
|------|--------|----------------|
| Left | 2 | 10 |
| Right | 3 | 7 |

## Build

```bash
cd /home/yctan/g1_motor_control/build
cmake -DUSE_MOCK_ECAT=ON ..  # For simulation
cmake ..                      # For real hardware
make ec_rt_thread
```

## Run

```bash
sudo ./bin/ec_rt_thread
```
