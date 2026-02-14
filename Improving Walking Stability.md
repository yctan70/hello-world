### Fix 1: joint_ids_map — Joint Ordering Mismatch

The policy and `MuJoCo` use different joint orderings:
```
MuJoCo/DDS order (anatomical):          Policy order (grouped by type):
  0: left_hip_pitch                       0 → DDS 0  (left_hip_pitch)
  1: left_hip_roll                        1 → DDS 6  (right_hip_pitch)
  2: left_hip_yaw                         2 → DDS 12 (waist_yaw)
  3: left_knee                            3 → DDS 1  (left_hip_roll)
  4: left_ankle_pitch                     4 → DDS 7  (right_hip_roll)
  5: left_ankle_roll                      5 → DDS 13 (waist_roll)
  6: right_hip_pitch                      ...
  ...
```
The `joint_ids_map = [0, 6, 12, 1, 7, 13, ...]` maps policy index → DDS/`MuJoCo` index. All observations and actions must go through this:

```python
# Reading joint states: MuJoCo → policy order (for observations)
q_policy = np.array([q_all[joint_ids_map[i]] for i in range(num_joints)])
# Writing target positions: policy order → MuJoCo (for control)
for policy_idx in range(num_joints):
    mj_data.ctrl[joint_ids_map[policy_idx]] = target_pos_policy[policy_idx]
```
Without this, left-leg actions would be applied to right-leg joints, waist actions to arms, etc. — instant crash.

### Fix 2: PD Gain Ordering (The Critical Bug)

This was the bug that caused walking instability. In `articulation.h`, the fields are explicitly commented:
```cpp
std::vector<float> joint_stiffness; // sdk order   ← DDS/MuJoCo order!
std::vector<float> joint_damping;   // sdk order   ← DDS/MuJoCo order!
```
The `stiffness` array in `deploy.yaml`:
```yaml
stiffness: [100, 100, 100, 150, 40, 40,   # L_hip(3), L_knee, L_ankle(2)
            100, 100, 100, 150, 40, 40,   # R_hip(3), R_knee, R_ankle(2)
            200, 200, 200,                 # waist(3) — stiffest!
            40, 40, 40, 40, 40, 40, 40,   # L_arm(7)
            40, 40, 40, 40, 40, 40, 40]   # R_arm(7)
```
This is clearly **DDS/`MuJoCo` actuator order** (symmetric L/R, anatomical grouping). But `default_joint_pos` is in policy order (confirmed by the C++ update() function using `joint_ids_map` to read into `data.joint_pos`).
**The bug**: My original code treated stiffness as policy order and remapped:
```python
# WRONG — stiffness is already in MuJoCo order!
for pi in range(num_joints):
    mi = joint_ids_map[pi]
    kps_mj[mi] = stiffness[pi]  # scrambles the gains!
```    
What this caused:
```
Waist joints (MuJoCo idx 12,13,14):
  Should get: kp = [200, 200, 200]  (from stiffness[12,13,14])
  Actually got: kp = [100, 40, 100]  (from stiffness[2,5,8] via mapping)
``` 
The waist — which supports the entire upper body — was only half as stiff as it should be. The fix was trivial:
```python
# CORRECT — use directly, no remapping needed!
kps_mj = np.array(cfg["stiffness"]) * kp_scale
kds_mj = np.array(cfg["damping"]) * kd_scale
```

### Fix 3: Motor Actuators → Position Actuators

On the real robot, you can see in the bridge code you're looking at (`unitree_sdk2py_bridge.py`:111-123):
```python
# Bridge converts motor commands to MuJoCo torques
self.mj_data.ctrl[i] = (
    msg.motor_cmd[i].tau
    + msg.motor_cmd[i].kp * (msg.motor_cmd[i].q - self.mj_data.sensordata[i])
    + msg.motor_cmd[i].kd * (msg.motor_cmd[i].dq - self.mj_data.sensordata[i + num_motor])
)
```
The real motor driver receives (`q_target`, `kp`, `kd`, `dq_target`, `tau`) and runs this PD internally at ~10 kHz. The `MuJoCo` model uses `<motor>` actuators where `ctrl` = torque.
Instead of computing PD manually at 500 Hz (1 torque per physics step), I convert the actuators programmatically to position type:
```python
def convert_to_position_actuators(model, kps_mj, kds_mj):
    for i in range(model.nu):
        # MuJoCo general actuator: force = gain*ctrl + bias[0] + bias[1]*q + bias[2]*dq
        # Setting: gain=kp, bias=[0, -kp, -kd]
        # → force = kp*ctrl + (-kp)*q + (-kd)*dq = kp*(ctrl - q) - kd*dq
        model.actuator_gainprm[i, 0] = kps_mj[i]
        model.actuator_biasprm[i] = [0, -kps_mj[i], -kds_mj[i]]
```
Now `ctrl[i]` = target position and `MuJoCo` evaluates PD at every physics step with the latest joint state — exactly like the real motor driver.

### Stability Improvements

Feature |	Default |	Effect
--------|-----------|------------
PD gain scale |	`kp×2.0` |	Compensates for MuJoCo physics gap
Ground friction |	1.5 |	Better foot traction during walking
Action EMA |	0.3 |	Smooths policy output to reduce jitter

### Verification Results

Walking (`vx=0.5`): Stable for 15+ seconds, height maintains 0.78-0.79, forward velocity ~0.5 m/s.
```
t=0.0s | h=0.791 | pos=[0.00,-0.00] | grav=[0.00,-0.00,-1.00]
t=3.0s | h=0.785 | pos=[1.37,0.02]  | grav=[0.04,-0.04,-1.00]
t=5.0s | h=0.787 | pos=[2.38,0.14]  | grav=[0.04,-0.05,-1.00]
t=7.0s | h=0.777 | pos=[3.39,0.33]  | grav=[0.07,0.02,-1.00]
```

### Usage

```bash
source unitree_mujoco/.venv/bin/activate
python unitree_mujoco/g1_rl_inference.py
python unitree_mujoco/g1_rl_inference.py --vx 0 --vy 0     # stand
python unitree_mujoco/g1_rl_inference.py --kp-scale 3.0     # stiffer
python unitree_mujoco/g1_rl_inference.py --debug             # diagnostics
```
