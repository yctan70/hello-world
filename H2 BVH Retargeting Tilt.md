# H2 BVH Retargeting — Tilt/Roll Debug Notes

## Problem Statement

When retargeting LAFAN1 walking motion to the Topstar H2, the robot continuously pitched forward and rolled laterally during playback. The base would accumulate tilt over time, and the `waist_roll` joint would drift to its ±39° hardware limit during walking strides.

---

## Root Cause Analysis

### 1. Structural difference: H2 lacks a waist pitch joint

Robots like Unitree G1 and PAL Talos have a `waist_pitch` joint between the pelvis and torso. This joint absorbs forward/backward lean from the human pelvis motion. H2 only has `waist_yaw` and `waist_roll` — no pitch. Without this joint, any forward lean in the human data propagates directly into base_link pitch via IK.

**Fix**: `root_yaw_only: true` in the IK config. The `_extract_yaw_only()` method strips pitch and roll from the human Hips orientation, giving the IK solver a purely upright + yaw target for the base. Combined with a high base_link rotation weight (200/300), this keeps the base level.

### 2. Base orientation weight too low to resist foot constraints

In `ik_match_table2`, the base_link orientation weight (originally 5, then 10, then 100) was losing to the combined foot position constraints (2 × 50 = 100). The IK solver would satisfy foot positions by tilting the base.

**Fix**: base_link rot_weight raised to **200** in table1 and **300** in table2.

### 3. Shoulder orientation constraints were driving waist_roll to joint limits

The human arm swing during walking causes shoulder orientations to rotate significantly. With shoulder `rot_weight = 100` in both tables, the IK strongly tried to match these orientations. Since H2 has no dedicated shoulder abduction chain independent of the torso, the solver deflected the `waist_roll` joint (the nearest lateral DOF) to ±39° to partially satisfy shoulder orientation targets.

This was the **primary cause** of the waist_roll limit-hitting during walking. It was not the pelvis motion — it was the arms.

**Fix**: shoulder rot_weight reduced from **100 → 30** in both tables (left and right `shoulder_roll_Link`).

### 4. Over-constraining waist_roll with posture regularization caused base to absorb lateral load

With `waist_roll_joint` posture weight = 500 (locking it near 0°), the IK had no joint to accommodate the slight lateral motions of the human. The base_link rolled instead, accumulating to -14° over 170 frames of walking.

**Fix**: waist_roll posture weight reduced to **150** (allows ±1–2° natural movement without hitting limits).

### 5. Hip roll joints over-constrained, preventing natural gait

With hip_roll posture weight = 100–500, the hip joints couldn't perform their natural abduction/adduction during walking (±5–10°). This suppressed natural gait kinematics.

**Fix**: hip_roll posture weight reduced to **30** (allows natural ±10° abduction while preventing runaway drift).

---

## Final Configuration (`bvh_lafan1_to_h2.json`)

| Parameter | Before | After |
|-----------|--------|-------|
| `root_yaw_only` | false | **true** |
| base_link rot_weight (table1) | 10 | **200** |
| base_link rot_weight (table2) | 5 | **300** |
| shoulder rot_weight (both tables) | 100 | **30** |
| waist_roll posture weight | 500 | **150** |
| hip_roll posture weight | 100 | **30** |
| waist_yaw posture weight | 200 | 200 (unchanged) |
| human_scale_table (all limbs) | 1.0 | **0.92** (legs/torso), **0.75** (arms) |

### Diagnostic results on walk1_subject2.bvh (300 frames)

| Metric | Problematic | Fixed |
|--------|-------------|-------|
| base_roll | up to **-14°** (accumulating) | **≤ ±0.3°** |
| base_pitch | up to **+3°** | **≤ ±0.2°** |
| waist_roll | hitting **±39° limit** | **≤ ±1.5°** |
| hip_roll | near 0° (locked) | **±9.5°** (natural gait) |

---

## Known Limitation: Body Tilt for Dance/Expressive Motion

The fix works by forcing the base_link to stay upright (`root_yaw_only` strips pitch/roll from Hips target, high rotation weights enforce this). This is correct for locomotion (walking/running) where an upright torso is expected.

**However, it breaks expressive full-body motions** (dance moves, bow, side lean) where the human intentionally tilts the whole body. In those cases:

- The IK solver still constrains base_link to be upright (because `root_yaw_only` strips the tilt from the Hips target).
- The hip_roll/knee joints cannot fully compensate for large intended tilts.
- The robot appears "stiff" and cannot replicate body-lean choreography.

### Potential solutions for dance/expressive motion

1. **Per-motion-type config**: Keep the current config for walking/running. Create a separate `bvh_lafan1_to_h2_dance.json` with:
   - `root_yaw_only: false` (allow full Hips orientation tracking)
   - base_link rot_weight lowered (e.g., 50/50) to let the body tilt
   - Higher waist_roll and hip_roll posture weights to still prevent runaway drift
   - Velocity limits on joints to prevent abrupt jumps

2. **Adaptive root constraint**: Dynamically detect whether the source motion has large intended hip tilt (> threshold) and blend between yaw-only and full orientation constraint.

3. **Waist pitch workaround**: Since H2 has no waist_pitch joint, intentional forward/backward lean must be entirely expressed through hip_pitch joints. The hip_pitch joints would need low posture regularization and the IK would need a FrameTask on the torso (waist_roll_Link) with a forward-lean target derived from human Spine2 rather than Hips.

---

## Files Modified

- `general_motion_retargeting/motion_retarget.py`
  - Added `root_yaw_only` config flag loading
  - Added `_extract_yaw_only()` method
  - Applied yaw-only extraction in `update_targets()` for root body
  - Added safety check in `offset_human_data()` for bodies not in offset tables
  - Increased `max_iter`: 10 → 20

- `general_motion_retargeting/ik_configs/bvh_lafan1_to_h2.json`
  - All changes listed in the table above

