# Fix Wave-Hand Arm Twist When Using MIT Mode

## Symptom

After migrating LS motors from **CST mode** to **MIT mode**, the `--wave_hand` gesture
caused the right arm to twist into an unexpected position at the start of the motion
before recovering into the correct waving pose.  The motion was correct in CST mode.

## Root Cause

The mock EtherCAT library (`src/ecrt_mock.c`) was written for the original CST mode
PDO layout and was never updated when the LS motor PDO was changed to MIT mode.

### CST vs MIT mode PDO layout

| Object | CST mode | MIT mode |
|--------|----------|----------|
| Torque output   | `0x6071` (target torque — host computes impedance) | `0x60B2` (feedforward torque only) |
| Position target | *not in PDO* | `0x607A` (target position) |
| Gain registers  | *not in PDO* | `0x2780,01` (KP) / `0x2780,02` (KD) |

**In CST mode** the *host* (`ec_rt_thread`) computed impedance:
```
τ = KP·(q_des − q) + KD·(0 − dq)
```
and sent the result as target torque via `0x6071`.  The mock tracked this register
and reproduced the correct restoring force in simulation.

**In MIT mode** the *motor itself* computes impedance from `q_des`, `KP`, `KD`
received over the PDO.  The mock was unaware of `0x607A` and `0x2780`; it still
tried to read `off_target_torque` which was never assigned and defaulted to byte
offset 0 in the PDO domain buffer.  That byte-0 location holds the **control word**
(0x000F = 15 when enabled), so every LS motor received a constant +0.015 Nm drive
torque with no restoring force.

### Drift accumulation

During the ~5 seconds the robot executes `stand_up`, each arm joint drifted
approximately +0.75 rad in the mock without any resistance.  The MuJoCo viewer
(driven by the sim bridge) held the arm correctly at 0 rad because it applied the
FSM's `hold_upper_body` commands correctly.

### Visible twist

When `wave_hand` starts, `h2_arm_task_update` snapshots the *mock* motor positions
(≈ +0.75 rad) as the ramp-in starting point.  The sim bridge then publishes
`q_des = +0.75 rad, kp = 40` to MuJoCo, which applies roughly 30 Nm in the wrong
direction — producing the visible twist — before the ramp trajectory drives the arm
toward the correct `WAVE_HAND_POSE`.

## Fix — `src/ecrt_mock.c`

Three changes were made:

### 1. Extended `ls_pdo_offsets_t`

Added fields to track MIT-specific PDO offsets and a flag to select the physics path:

```c
typedef struct {
  unsigned int off_control_word;
  unsigned int off_target_torque;   // 0x6071 (old CST) or 0x60B2 (MIT tff)
  unsigned int off_target_position; // 0x607A (MIT)
  unsigned int off_kp;              // 0x2780,01 (MIT)
  unsigned int off_kd;              // 0x2780,02 (MIT)
  unsigned int off_mode;
  unsigned int off_status_word;
  unsigned int off_mode_display;
  unsigned int off_position;
  unsigned int off_velocity;
  unsigned int off_torque;
  int has_mit_pdos;
} ls_pdo_offsets_t;
```

### 2. Track MIT registers in `ecrt_domain_reg_pdo_entry_list`

Added cases for `0x60B2`, `0x607A`, and `0x2780` (with correct 2-byte sizes).
Setting `has_mit_pdos = 1` when `0x607A` is registered selects the MIT physics
path at runtime, preserving backward compatibility with any code still using `0x6071`.

### 3. MIT impedance physics in `ecrt_master_send`

When `has_mit_pdos` is set, the mock now computes impedance locally — mimicking
what the real LS motor firmware does:

```c
float pos_des = target_pos_raw / 10000.0f;  // encoder counts → radians
float kp = (float)kp_raw;                   // to_kp = 1.0 (kp_max=65535)
float kd = (float)kd_raw;
float tff = tff_raw / 1000.0f;              // permille → Nm

motor->torque = kp * (pos_des - pos_actual)
              + kd * (0.0f   - vel_actual)
              + tff;
```

Scale factors match the SDO mock values in `ecrt_master_sdo_upload`:
- `0x6092,01 = 62832` → `increment_to_rad = 2π/62832 ≈ 1/10000`
- `0x2781,01 = 65535` → `to_kp = 65535/65535 = 1.0`
- `0x2790,05 = 1000`  → `permille_to_torque = (1.0 Nm)/1000`

## Files Changed

| File | Change |
|------|--------|
| `src/ecrt_mock.c` | Extended `ls_pdo_offsets_t`; added MIT offset tracking in `ecrt_domain_reg_pdo_entry_list`; implemented MIT impedance physics in `ecrt_master_send` |

