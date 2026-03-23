---


---

<h1 id="topstar-h2-—-robot-overview--locomotion-status">Topstar H2 — Robot Overview &amp; Locomotion Status</h1>
<h2 id="hardware">Hardware</h2>
<ul>
<li><strong>Platform:</strong> Topstar H2 humanoid robot</li>
<li><strong>DOF:</strong> 30 total
<ul>
<li>Left leg: 6 (Hip Pitch/Roll/Yaw, Knee, Ankle Pitch/Roll)</li>
<li>Right leg: 6 (same)</li>
<li>Waist: 2 (Yaw, Roll)</li>
<li>Head: 2 (Yaw, Pitch)</li>
<li>Left arm: 7 (Shoulder Pitch/Roll/Yaw, Elbow, Wrist Yaw/Pitch/Roll)</li>
<li>Right arm: 7 (same)</li>
</ul>
</li>
<li><strong>Hands:</strong> LS Hand Pro dexterous hands (12 pressure/force sensors each)</li>
<li><strong>IMU:</strong> Yesense (460800 baud serial)</li>
<li><strong>Motor buses:</strong> 4 EtherCAT masters (SE motors for legs, LS/CiA402 for waist/head/arms)</li>
<li><strong>Controller:</strong> EtherCAT RT loop at 1 kHz</li>
</ul>
<h2 id="codebase-at-topstar_h2">Codebase at <code>~/topstar_h2</code></h2>
<h3 id="architecture">Architecture</h3>
<pre><code>ec_rt_thread (1 kHz RT loop, EtherCAT)
    ↓ shared memory IPC (/dev/shm/ec_motor_shm)
topstar_bridge (user-space DDS bridge)
    ↓ CycloneDDS topics
User applications (Python, ROS2, SDK2)
</code></pre>
<h3 id="key-source-files">Key Source Files</h3>

<table>
<thead>
<tr>
<th>File</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>src/ec_rt_thread.c</code></td>
<td>Main real-time control loop</td>
</tr>
<tr>
<td><code>src/h2_fsm.c</code></td>
<td>FSM: ZeroTorque → Damp → Squat → Sit → StandUp → Locomotion / RLInference</td>
</tr>
<tr>
<td><code>src/h2_gait.c</code></td>
<td><strong>CPG gait generator</strong> (sinusoidal, velocity-commanded)</td>
</tr>
<tr>
<td><code>src/h2_balance.cpp</code></td>
<td><strong>Pinocchio balance controller</strong> (gravity comp + IMU PID)</td>
</tr>
<tr>
<td><code>src/h2_arm_task.c</code></td>
<td>Arm gesture/task control</td>
</tr>
<tr>
<td><code>src/ls_hand.c</code></td>
<td>LS dexterous hand EtherCAT control</td>
</tr>
<tr>
<td><code>src/imu_serial.c</code></td>
<td>Yesense IMU driver</td>
</tr>
<tr>
<td><code>src/topstar_bridge.cpp</code></td>
<td>DDS bridge (publishes <code>rt/lowstate</code>, <code>rt/lowcmd</code>, hand topics)</td>
</tr>
<tr>
<td><code>python/h2_rl_runner.py</code></td>
<td>ONNX RL policy inference runner</td>
</tr>
<tr>
<td><code>model/urdf/h2.xml</code></td>
<td>Full MuJoCo model (TH010-500-URDF-20260310)</td>
</tr>
</tbody>
</table><h2 id="what-you-can-do-right-now-no-rl-policy-required">What You Can Do Right Now (No RL Policy Required)</h2>

<table>
<thead>
<tr>
<th>Capability</th>
<th>How</th>
<th>Status</th>
</tr>
</thead>
<tbody>
<tr>
<td>Standing + balance</td>
<td>FSM <code>StandUp</code> state + <code>h2_balance.cpp</code> (Pinocchio RNEA + IMU PID)</td>
<td>Ready</td>
</tr>
<tr>
<td>Walking</td>
<td>FSM <code>Locomotion</code> state → <code>h2_gait.c</code> CPG</td>
<td>Ready</td>
</tr>
<tr>
<td>Teleoperation</td>
<td>PS2 joystick → velocity commands <code>(vx, vy, omega)</code></td>
<td>Ready</td>
</tr>
<tr>
<td>Arm / hand control</td>
<td><code>h2_arm_task.c</code>, <code>ls_hand.c</code>, DDS <code>ArmActionServer</code></td>
<td>Ready</td>
</tr>
<tr>
<td>MuJoCo simulation</td>
<td><code>mujoco_sim_bridge</code> (standalone or digital-twin mirror mode)</td>
<td>Ready</td>
</tr>
<tr>
<td>Motor GUI</td>
<td><code>python/motor_gui.py</code> (PySide6)</td>
<td>Ready</td>
</tr>
</tbody>
</table><h2 id="what-requires-a-trained-rl-policy">What Requires a Trained RL Policy</h2>

<table>
<thead>
<tr>
<th>Capability</th>
<th>Blocker</th>
</tr>
</thead>
<tbody>
<tr>
<td>FSM <code>RLInference</code> state</td>
<td>Needs <code>.onnx</code> policy file</td>
</tr>
<tr>
<td>Robust disturbance rejection</td>
<td>CPG is open-loop; RL provides feedback robustness</td>
</tr>
<tr>
<td>Dynamic locomotion (running, uneven terrain)</td>
<td>CPG is limited to steady-state walking</td>
</tr>
<tr>
<td>Whole-body loco-manipulation</td>
<td>Requires unified or hierarchical RL policy</td>
</tr>
</tbody>
</table><h2 id="recommended-path-to-rl-locomotion">Recommended Path to RL Locomotion</h2>
<ol>
<li><strong>Use CPG gait now</strong> — walking works today via <code>h2_gait.c</code>.</li>
<li><strong>Train RL policy in Isaac Lab:</strong>
<ul>
<li>Import <code>model/urdf/h2.xml</code> as the robot asset.</li>
<li>Adapt a Unitree G1 locomotion task to H2’s 30-DOF config (<code>src/h2_robot_config.h</code>).</li>
<li>Domain randomize: mass, friction, motor damping, action delay.</li>
</ul>
</li>
<li><strong>Export to ONNX</strong> and point <code>python/h2_rl_runner.py</code> at the policy file.</li>
<li><strong>Layer manipulation on top</strong> via DDS <code>ArmActionServer</code> or a separate arm policy.</li>
<li><strong>Optional:</strong> Register H2 as a new embodiment in Isaac-GR00T for VLA-based task control.</li>
</ol>
<h2 id="build-quick-reference">Build Quick Reference</h2>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Real hardware</span>
<span class="token function">sudo</span> ./build/bin/ec_rt_thread
./build/bin/topstar_bridge --network_interface<span class="token operator">=</span>enp3s0

<span class="token comment"># MuJoCo simulation</span>
./run_mujoco_viewer.sh
./run_mujoco_sim.sh

<span class="token comment"># RL inference (once policy is available)</span>
python3 python/h2_rl_runner.py --lab-path /path/to/topstar_rl_lab

<span class="token comment"># Mock EtherCAT (no hardware/sim)</span>
cmake -DUSE_MOCK_ECAT<span class="token operator">=</span>ON <span class="token punctuation">..</span>. <span class="token operator">&amp;&amp;</span> <span class="token function">sudo</span> ./build/bin/ec_rt_thread
</code></pre>

