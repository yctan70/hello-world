# CycloneDDS Message Modifications

This document describes the modifications made to the `cyclonedds_ws` messages to support only the G1 robot type and replace the Dex3 hand with the LS (Leadshine) hand.

## Overview

The CycloneDDS message definitions were modified to:
1. Remove `unitree_go` package and all Go/quadruped robot messages
2. Create new LS hand message definitions
3. Update existing hand messages to use the LS hand structure

---

## New LS Hand Messages

### HandMotorCmd.msg

Command message for individual LS hand motor control:

```
int16 target_position    # Target position (encoder counts / 100)
int16 target_velocity    # Target velocity
int16 max_current        # Maximum current limit (mA)
uint8 mode               # Control mode (0=position, 1=velocity)
uint8 reserve            # Reserved
```

### HandMotorState.msg

State feedback from individual LS hand motor:

```
int16 position           # Current position (encoder counts / 100)
int16 velocity           # Current velocity
int16 current            # Current draw (mA)
uint8 status             # Motor status flags
uint8 error              # Error code
```

---

## Updated Hand Messages

### HandCmd.msg

Complete hand command message (6-DOF LS hand):

```
HandMotorCmd[6] motor_cmd   # Commands for 6 motors
uint8 control_mode          # 0=position mode
uint8 enable                # 1=enable, 0=disable
uint8 home                  # 1=trigger homing
uint8 reserve               # Reserved
uint32 crc                  # CRC32 checksum
```

### HandState.msg

Complete hand state feedback:

```
HandMotorState[6] motor_state   # State for 6 motors
PressSensorState press_sensor   # Pressure sensor data
float32[12] normal_force        # Normal force per sensor
float32[12] tangential_force    # Tangential force per sensor
float32[12] proximity           # Proximity sensor values
float32 power_v                 # Power supply voltage
float32 power_a                 # Power supply current
uint32[2] error                 # Error codes
uint32[2] reserve               # Reserved
uint8 hand_id                   # Hand ID (0=left, 1=right)
uint8 dof_count                 # Number of DOF (6)
uint8 operational               # 1=operational state
uint8 homed                     # 1=homing complete
uint32 crc                      # CRC32 checksum
```

### PressSensorState.msg

Pressure sensor state:

```
float32[12] pressure       # Pressure readings per sensor
float32[12] temperature    # Temperature readings per sensor
uint32 lost                # Lost packet count
uint32 reserve             # Reserved
```

---

## Package Structure

```
cyclonedds_ws/src/unitree/
├── unitree_hg/           # Humanoid/G1 messages (kept)
│   └── msg/
│       ├── BmsCmd.msg
│       ├── BmsState.msg
│       ├── HandCmd.msg         # Updated for LS hand
│       ├── HandMotorCmd.msg    # NEW - LS hand motor command
│       ├── HandMotorState.msg  # NEW - LS hand motor state
│       ├── HandState.msg       # Updated for LS hand
│       ├── IMUState.msg
│       ├── LowCmd.msg
│       ├── LowState.msg
│       ├── MainBoardState.msg
│       ├── MotorCmd.msg
│       ├── MotorState.msg
│       └── PressSensorState.msg
└── unitree_api/          # API messages (kept)
```

Note: `unitree_go` package was removed (quadruped robot messages not needed for G1).

---

## Build Instructions

```bash
cd ~/unitree_ros2/cyclonedds_ws
colcon build

# Then rebuild the example workspace
cd ~/unitree_ros2/example
source ../cyclonedds_ws/install/local_setup.bash
colcon build
```

---

## DDS Communication

The bridge (`unitree_bridge`) uses these messages on the following topics:

| Topic | Direction | Message Type |
|-------|-----------|--------------|
| `rt/hand/left/cmd` | Subscribe | HandCmd |
| `rt/hand/left/state` | Publish | HandState |
| `rt/hand/right/cmd` | Subscribe | HandCmd |
| `rt/hand/right/state` | Publish | HandState |

The ROS2 examples use topic names without `rt/` prefix (ROS2 rmw_cyclonedds adds it automatically).

