# DDS Latency Investigation

**Date:** 2026-03-05
**Context:** Comparing latency of `~/g1_motor_control/src/unitree_bridge.cpp` (custom EtherCAT bridge) against `~/unitree_rl_lab/deploy/robots/g1_29dof` (reference RL controller `g1_ctrl`) for RL inference control of the G1 robot.

---

## Summary

Direct shared memory access is faster than the current DDS bridge because the bridge introduces an extra IPC hop, a 2 ms polling delay, and `POSIX` semaphore contention with the RT EtherCAT thread. The dominant sources are identified below with specific fixes.

---

## Architecture Comparison

### `g1_ctrl` (reference — low latency)

```
Unitree firmware (1kHz EtherCAT)
  → DDS publish rt/lowstate
    → SubscriptionBase listener callback fires immediately
      → mutex copy into msg_
        → RecurrentThread (1ms) fires:
            pre_run()  → lowstate->update() (joystick parse only)
            run()      → RL inference, fill lowcmd->msg_
            post_run() → unlockAndPublish()
              → RealTimePublisher thread wakes (turn_ flag, no sleep)
                → CRC → DDS publish rt/lowcmd
                  → Unitree firmware applies to motors
```

- **Event-driven throughout**: no `sleep_for`, no polling
- **Single DDS hop** per direction
- `SubscriptionBase::InitChannel()` registers a DDS listener — callback fires at packet arrival with zero polling latency
- `RealTimePublisher::publishingLoop()` wakes immediately on `unlockAndPublish()` via atomic `turn_` flag

Relevant sources:
- `unitree_sdk2/include/unitree/dds_wrapper/common/Subscription.h:36–43`
- `unitree_sdk2/include/unitree/dds_wrapper/common/Publisher.h:133–162`
- `deploy/include/FSM/CtrlFSM.h:54–56` — `RecurrentThread` at 1 ms
- `deploy/robots/g1_29dof/src/FSMState.cpp:89–98` — `pre_run`/`post_run`

---

### unitree_bridge (custom — higher latency)

```
EtherCAT RT thread (1kHz)
  → writes to POSIX shared memory (sem_post)
    → bridge_thread wakes every 2ms (sleep_for)
      → sem_wait (may block if RT thread holds lock)
        → memcpy 29 motors + IMU + joystick → LowState_ struct
          → dds_write(lowstate_writer)
            → g1_ctrl receives, runs RL inference
              → dds publish rt/lowcmd
                → bridge_thread polls dds_take() at next 2ms wake
                  → sem_wait (blocks if RT thread holds lock)
                    → copy commands to shared memory
                      → RT thread reads on next EtherCAT cycle (≤1ms)
```

- **Polling-based**: bridge sleeps 2 ms between cycles
- **Double DDS hop** per control cycle (state out + command in)
- **Extra `SHM`↔DDS boundary** on both sides
- `dds_take()` polled in same 2 ms loop — commands wait up to 2 ms to be processed

Relevant sources:
- `g1_motor_control/src/unitree_bridge.cpp:527–576` — `run()` loop
- `g1_motor_control/src/unitree_bridge.cpp:572` — `sleep_for(2ms)`
- `g1_motor_control/src/unitree_bridge.cpp:705–745` — `process_lowcmd()` polling

---

## Root Causes of Latency

### 1. `sleep_for(2ms)` polling — dominant source

**Location:** `unitree_bridge.cpp:572`

The bridge thread sleeps 2 ms between iterations. A `rt/lowcmd` arriving from `g1_ctrl` sits undelivered to shared memory for up to 2 ms. Combined with the RL control loop at 1 ms in `g1_ctrl`, this means the full state→command round-trip latency is:

```
lowstate publish (≤2ms bridge cycle)
  + RL inference (<1ms)
  + lowcmd delivery (≤2ms bridge cycle)
  + EtherCAT cycle (≤1ms)
= up to ~6ms worst-case
```

`g1_ctrl` on native Unitree firmware achieves ~1–2 ms round-trip.

### 2. POSIX semaphore contention with RT thread

**Location:** `ec_shared_mem.h` — `lock_semaphore()` / `unlock_semaphore()`

`sem_wait()` is a blocking kernel `syscall`. When the bridge thread holds the semaphore, the RT EtherCAT thread must wait, jeopardizing the 1 kHz cycle. Conversely, the RT thread holding the semaphore delays the bridge. This mutual blocking creates:
- **Priority inversion risk**: low-priority bridge thread can delay high-priority RT thread
- **Jitter** in both RT cycle time and bridge publish rate

### 3. `dds_take()` polling vs. listener callbacks

**Location:** `unitree_bridge.cpp:714` (`process_lowcmd`), `unitree_bridge.cpp:668` (`poll_arm_sdk_cmd`)

`g1_ctrl` uses `ChannelSubscriber::InitChannel()` which registers a DDS **listener** — the middleware calls the handler the moment a packet is received, with no polling overhead. Your bridge uses `dds_take()` in a 2ms polling loop. A command arriving 0.1ms after a `dds_take()` call waits 1.9ms for the next poll.

### 4. `std::cout` on every received command

**Location:** `unitree_bridge.cpp:738`

```cpp
std::cout << "[LowCmd] Received command (mode_pr=...)" << std::endl;
```

At 500Hz bridge rate this is 500 synchronous `write()` syscalls per second. Each `std::endl` flushes the buffer. This causes measurable jitter in the bridge thread, delaying both state publishing and command processing.

### 5. Architectural extra hop

Even with all above fixes, the bridge architecture is inherently longer than native:

| Path | Hops |
|------|------|
| `g1_ctrl` + Unitree firmware | EtherCAT → DDS → RL → DDS → EtherCAT |
| `g1_ctrl` + unitree_bridge | EtherCAT → SHM → DDS → RL → DDS → SHM → EtherCAT |
| RL direct SHM access | EtherCAT → SHM → RL → SHM → EtherCAT |

Direct `SHM` eliminates both DDS hops and both `SHM`↔DDS copies, which is why it outperforms the bridge.

---

## Recommended Fixes

### Fix 1: Replace `sleep_for` with DDS `waitset` (eliminates 2 ms polling delay)

Replace the polling loop with a DDS `waitset` that blocks until `rt/lowcmd` arrives or a timeout elapses. The bridge publishes state, then immediately processes any pending command — no periodic sleep.

```cpp
// In initialize(), after creating lowcmd_reader:
dds_entity_t waitset = dds_create_waitset(participant);
dds_entity_t lowcmd_cond = dds_create_readcondition(
    lowcmd_reader, DDS_ANY_STATE);
dds_waitset_attach(waitset, lowcmd_cond, 0);

// In run(), replace sleep_for:
while (running) {
    // Read sensors, publish lowstate
    read_imu();
    read_joystick();
    publish_lowstate();

    // Block until lowcmd arrives or 1ms elapses
    dds_waitset_wait(waitset, NULL, 0, DDS_MSECS(1));

    // Process whatever arrived
    process_lowcmd();
    poll_arm_sdk_cmd();
    publish_hand_state(HAND_LEFT);
    publish_hand_state(HAND_RIGHT);
    process_hand_cmd(HAND_LEFT);
    process_hand_cmd(HAND_RIGHT);
}
```

This wakes the bridge **immediately** when `rt/lowcmd` arrives, matching the callback behavior of `SubscriptionBase`.

### Fix 2: Replace POSIX semaphore with `seqlock` (non-blocking RT thread)

The `seqlock` pattern lets the RT thread write without ever blocking on the bridge thread, and lets the bridge retry reads atomically without kernel calls.

```c
// Add to SharedMemoryData (or as a standalone atomic):
volatile uint32_t seq;  // odd = RT writing, even = data stable

// RT thread writer (never blocks, no kernel calls):
shm->seq++;                   // make odd: readers see write in progress
__sync_synchronize();
// ... write motor states ...
__sync_synchronize();
shm->seq++;                   // make even: data is now stable

// Bridge thread reader (retries only if caught mid-write):
uint32_t s;
do {
    s = shm->seq;
    if (s & 1) { continue; }  // RT is writing, spin briefly
    __sync_synchronize();
    // ... copy motor data ...
    __sync_synchronize();
} while (shm->seq != s);      // retry if seq changed during copy
```

This eliminates the semaphore entirely for the state-read path, removing all RT thread blocking.

### Fix 3: Remove `std::cout` from hot-path functions

In `process_lowcmd()`, `poll_arm_sdk_cmd()`, and `publish_hand_state()` — remove or gate per-packet logging. Log only on state transitions:

```cpp
// Before (logs every packet):
std::cout << "[LowCmd] Received command ..." << std::endl;

// After (log only on mode change):
if (cmd->mode_pr != last_mode_pr) {
    fprintf(stderr, "[LowCmd] mode_pr changed to %d\n", cmd->mode_pr);
    last_mode_pr = cmd->mode_pr;
}
```

### Fix 4: Elevate bridge thread scheduling priority

The bridge thread competes with other user-space threads for CPU time. Setting `SCHED_FIFO` priority just below the RT EtherCAT thread ensures the bridge is scheduled promptly:

```cpp
// In start(), after launching bridge_thread:
struct sched_param sp;
sp.sched_priority = 40;  // RT EtherCAT thread should be ~80
pthread_setschedparam(bridge_thread.native_handle(), SCHED_FIFO, &sp);
```

### Fix 5 (architectural): Direct `SHM` access from RL controller

If minimum latency is the goal, modify `g1_ctrl` (or write a variant) to read/write shared memory directly instead of via DDS. This eliminates both DDS hops and both copy operations:
```
EtherCAT RT (1kHz) ─── shared memory ─── RL inference thread (1kHz)
```
The `unitree_articulation.h` `update()` method reads from `lowstate->msg_` — a thin wrapper that reads from `SHM` instead of a DDS-subscribed buffer would remove all IPC overhead beyond one memory read.

---

## Expected Latency After Fixes

| Scenario | State→RL latency | RL→Motor latency | Round-trip |
|----------|-----------------|------------------|------------|
| Current bridge | ≤2 ms | ≤2 ms + 1 ms EtherCAT | ~5–6 ms |
| Fix 1+2+3 (`waitset` + `seqlock`) | <0.1 ms | <0.1 ms + 1 ms EtherCAT | ~1–2 ms |
| Direct `SHM` (no bridge) | <0.05 ms | <0.05 ms + 1 ms EtherCAT | ~1–1.1 ms |
| `g1_ctrl` + Unitree firmware | <0.1 ms DDS | <0.1 ms DDS + 1 ms EtherCAT | ~1–2 ms |

Fix 1 (`waitset`) eliminates the dominant 2 ms polling delay. Fix 2 (`seqlock`) eliminates RT jitter from semaphore contention. Together they bring the bridge latency within ~0.1–0.2 ms of direct `SHM`.


