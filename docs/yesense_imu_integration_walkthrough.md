# Yesense IMU Integration - Walkthrough

This document describes the integration of the Yesense IMU sensor with the G1 Robot motor control system, enabling IMU data to be published via the `unitree_sdk2`-compatible DDS bridge.

## Overview

The integration consists of three main components:

1. **`imu_serial.c/h`** - Low-level serial driver that reads and parses Yesense protocol data
2. **Shared Memory** - `IMUData` structure in `ec_shared_mem.h` for inter-process communication
3. **`unitree_bridge.cpp`** - Reads IMU data, stores in shared memory, and publishes via DDS

```mermaid
graph LR
    A[Yesense IMU<br>/dev/ttyUSB0] -->|Serial 460800 baud| B[imu_serial.c]
    B -->|Parse Protocol| C[IMUReading]
    C -->|Copy| D[Shared Memory<br>IMUData]
    D -->|Read| E[unitree_bridge.cpp]
    E -->|DDS Publish| F[rt/lowstate<br>LowState_.imu_state]
```

---

## Component Details

### 1. IMU Serial Driver (`imu_serial.c/h`)

**Location:** [imu_serial.h](file:///home/yctan/g1_motor_control/src/imu_serial.h) | [imu_serial.c](file:///home/yctan/g1_motor_control/src/imu_serial.c)

**Purpose:** Reads raw bytes from the Yesense IMU serial port, parses the TLV-based protocol, and outputs sensor data in a normalized format.

#### Key Features

| Feature | Implementation |
|---------|---------------|
| Baud Rate | 460800 (configurable) |
| Protocol | Yesense binary TLV format |
| Buffering | Ring buffer for async reads |
| CRC Validation | 2-byte checksum verification |
| Unit Conversion | deg→rad for angles, deg/s→rad/s for gyro |

#### Data Output Structure

```c
typedef struct {
  float quaternion[4];    // w, x, y, z
  float gyroscope[3];     // rad/s
  float accelerometer[3]; // m/s²
  float rpy[3];           // rad (roll, pitch, yaw)
  int16_t temperature;    // Celsius * 100
  uint64_t timestamp_us;  // Sample timestamp
  bool valid;
} IMUReading;
```

#### Yesense Protocol Parsing

The driver parses these TLV data IDs:

| Data ID | Name | Length | Conversion |
|---------|------|--------|------------|
| `0x01` | Temperature | 2 bytes | × 0.01 |
| `0x10` | Accelerometer | 12 bytes | × 1e-6 m/s² |
| `0x20` | Gyroscope | 12 bytes | × 1e-6 deg/s → rad/s |
| `0x40` | Euler angles | 12 bytes | × 1e-6 deg → rad |
| `0x41` | Quaternion | 16 bytes | × 1e-6 |
| `0x51` | Timestamp | 4 bytes | microseconds |

---

### 2. Shared Memory Structure (`ec_shared_mem.h`)

**Location:** [ec_shared_mem.h](file:///home/yctan/g1_motor_control/src/ec_shared_mem.h)

The IMU data is stored in the `SharedMemoryData` structure alongside motor data:

```c
typedef struct {
  float quaternion[4];    // Orientation (w,x,y,z)
  float gyroscope[3];     // Angular velocity (rad/s)
  float accelerometer[3]; // Linear acceleration (m/s²)
  float rpy[3];           // Roll, Pitch, Yaw (rad)
  int16_t temperature;    // IMU temperature
  uint8_t padding[2];
  uint64_t timestamp_ns;  // Sample timestamp
  bool valid;             // Data valid flag
  uint8_t padding2[7];
} IMUData;
```

This structure is compatible with `unitree_sdk2`'s `IMUState_` format.

---

### 3. Unitree Bridge Integration (`unitree_bridge.cpp`)

**Location:** [unitree_bridge.cpp](file:///home/yctan/g1_motor_control/src/unitree_bridge.cpp)

#### Initialization

```cpp
// Initialize IMU serial port
imu_fd = imu_serial_init(IMU_DEFAULT_DEVICE, IMU_DEFAULT_BAUDRATE);
if (imu_fd < 0) {
  std::cerr << "Warning: IMU not available, continuing without IMU" << std::endl;
}
```

> [!NOTE]
> The bridge continues to operate even if the IMU is not connected, providing default values (identity quaternion).

#### Main Loop

The bridge runs at ~500 Hz and performs these operations each cycle:

1. **Read IMU data** from serial port
2. **Copy to shared memory** (with semaphore lock)
3. **Publish LowState** with motor + IMU data via DDS
4. **Process LowCmd** commands from DDS

```cpp
// Read IMU data into shared memory
if (imu_fd >= 0 && shm_data) {
  IMUReading reading;
  if (imu_serial_read(imu_fd, &reading) > 0 && reading.valid) {
    lock_semaphore();
    memcpy(shm_data->imu.quaternion, reading.quaternion, sizeof(reading.quaternion));
    memcpy(shm_data->imu.gyroscope, reading.gyroscope, sizeof(reading.gyroscope));
    memcpy(shm_data->imu.accelerometer, reading.accelerometer, sizeof(reading.accelerometer));
    memcpy(shm_data->imu.rpy, reading.rpy, sizeof(reading.rpy));
    shm_data->imu.temperature = reading.temperature;
    shm_data->imu.timestamp_ns = reading.timestamp_us * 1000;
    shm_data->imu.valid = true;
    unlock_semaphore();
  }
}
```

#### DDS Publishing

IMU data is included in the `LowState_` message:

```cpp
if (shm_data->imu.valid) {
  memcpy(state.imu_state.quaternion, shm_data->imu.quaternion, ...);
  memcpy(state.imu_state.gyroscope, shm_data->imu.gyroscope, ...);
  memcpy(state.imu_state.accelerometer, shm_data->imu.accelerometer, ...);
  memcpy(state.imu_state.rpy, shm_data->imu.rpy, ...);
  state.imu_state.temperature = shm_data->imu.temperature;
}
```

---

## Usage

### Running the System

```bash
# Terminal 1: Start RT thread (requires sudo for real-time permissions)
cd /home/yctan/g1_motor_control/build
sudo ./bin/ec_rt_thread

# Terminal 2: Start Unitree bridge (reads IMU and publishes DDS)
./bin/unitree_bridge
```

### Verifying IMU Data

Check that the IMU is detected during startup:

```
[IMU] Opened /dev/ttyUSB0 at 460800 baud
```

If not connected:

```
Warning: IMU not available, continuing without IMU
```

---

## File Summary

| File | Purpose |
|------|---------|
| [imu_serial.h](file:///home/yctan/g1_motor_control/src/imu_serial.h) | IMU driver interface |
| [imu_serial.c](file:///home/yctan/g1_motor_control/src/imu_serial.c) | Yesense protocol parser (380 lines) |
| [ec_shared_mem.h](file:///home/yctan/g1_motor_control/src/ec_shared_mem.h) | IMUData structure definition |
| [unitree_bridge.cpp](file:///home/yctan/g1_motor_control/src/unitree_bridge.cpp) | Integration with DDS publishing |
| [CMakeLists.txt](file:///home/yctan/g1_motor_control/src/CMakeLists.txt) | Build configuration (lines 130-157) |
