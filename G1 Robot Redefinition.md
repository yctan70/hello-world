---


---

<h1 id="g1-robot-redefinition---implementation-walkthrough">G1 Robot Redefinition - Implementation Walkthrough</h1>
<h2 id="summary">Summary</h2>
<p>Redefined the Unitree G1 humanoid robot (29 DOF) with custom EtherCAT hardware using 4 masters controlling 29 motors. The implementation follows unitree_sdk2 joint ID conventions.</p>
<h2 id="hardware-configuration">Hardware Configuration</h2>

<table>
<thead>
<tr>
<th>Master</th>
<th>Motor Type</th>
<th>Count</th>
<th>Body Parts</th>
</tr>
</thead>
<tbody>
<tr>
<td>0</td>
<td>SE</td>
<td>9</td>
<td>Left Leg (6) + Waist (3)</td>
</tr>
<tr>
<td>1</td>
<td>SE</td>
<td>6</td>
<td>Right Leg (6)</td>
</tr>
<tr>
<td>2</td>
<td>LS</td>
<td>7</td>
<td>Left Arm (7)</td>
</tr>
<tr>
<td>3</td>
<td>LS</td>
<td>7</td>
<td>Right Arm (7)</td>
</tr>
</tbody>
</table><h2 id="files-changed">Files Changed</h2>
<h3 id="new-files">New Files</h3>
<ul>
<li><code>src/g1_robot_config.h</code> - G1 joint definitions, body parts, master mappings</li>
<li><code>src/unitree_hg.idl</code> - unitree_sdk2 compatible LowCmd/LowState messages</li>
<li><code>src/unitree_bridge.cpp</code> - Bridge between shared memory and unitree_sdk2</li>
</ul>
<h3 id="modified-files">Modified Files</h3>
<ul>
<li><code>src/constants.h</code> - Updated for 4 masters, 29 motors</li>
<li><code>src/motors.h</code> - Expanded master context with motor type</li>
<li><code>src/ec_shared_mem.h</code> - 29-motor array with joint IDs</li>
<li><code>src/ec_rt_thread.c</code> - 4-master RT thread with G1 joint mapping</li>
<li><code>src/ecrt_mock.c</code> - 4-master mock simulation</li>
<li><code>src/MotorControl.idl</code> - Added body parts, joint names</li>
<li><code>src/dds_bridge.cpp</code> - Updated for 29 motors with joint names</li>
</ul>
<h2 id="key-design-decisions">Key Design Decisions</h2>
<ol>
<li>
<p><strong>Joint ID Convention</strong>: Follows unitree_sdk2 <code>G1JointIndex</code> ordering (left leg → right leg → waist → left arm → right arm)</p>
</li>
<li>
<p><strong>Motor Types</strong>:</p>
<ul>
<li>SE motors (simplified protocol) for legs and waist (masters 0-1)</li>
<li>LS motors (CiA 402 CST mode) for arms (masters 2-3)</li>
</ul>
</li>
<li>
<p><strong>Shared Memory Layout</strong>: Motors indexed by joint ID (0-28) for direct lookup</p>
</li>
<li>
<p><strong>Default Gains</strong>: Uses unitree_sdk2 default Kp/Kd values per joint</p>
</li>
</ol>
<h2 id="build-instructions">Build Instructions</h2>
<h3 id="with-mock-ethercat-simulation-mode">With Mock EtherCAT (simulation mode)</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span> build
<span class="token function">rm</span> -rf * <span class="token operator">&amp;&amp;</span> cmake -DUSE_MOCK_ECAT<span class="token operator">=</span>ON <span class="token punctuation">..</span>
<span class="token function">make</span> -j4
</code></pre>
<h3 id="with-real-ethercat-hardware">With Real EtherCAT Hardware</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">cd</span> build
<span class="token function">rm</span> -rf * <span class="token operator">&amp;&amp;</span> cmake <span class="token punctuation">..</span>
<span class="token function">make</span> -j4
</code></pre>
<p><strong>Build Targets:</strong></p>
<ul>
<li><code>bin/ec_rt_thread</code> - Real-time EtherCAT control</li>
<li><code>bin/dds_bridge</code> - DDS communication bridge</li>
<li><code>bin/unitree_bridge</code> - unitree_sdk2 compatible bridge</li>
<li><code>lib/libec_shared_mem.so</code> - Shared memory library</li>
</ul>
<h2 id="unitree_sdk2-integration">unitree_sdk2 Integration</h2>
<h3 id="message-structures-matching-unitree_hgmsgdds_">Message Structures (matching unitree_hg::msg::dds_)</h3>

<table>
<thead>
<tr>
<th>Structure</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>MotorCmd_</td>
<td>Per-motor command: mode, q, dq, tau, kp, kd</td>
</tr>
<tr>
<td>MotorState_</td>
<td>Per-motor state: q, dq, ddq, tau_est, temperature</td>
</tr>
<tr>
<td>IMUState_</td>
<td>IMU: quaternion, gyroscope, accelerometer, rpy</td>
</tr>
<tr>
<td>LowCmd_</td>
<td>Full robot command (35 motors)</td>
</tr>
<tr>
<td>LowState_</td>
<td>Full robot state (35 motors + IMU)</td>
</tr>
</tbody>
</table><h3 id="topic-names">Topic Names</h3>
<ul>
<li><code>rt/lowcmd</code> - Subscribe for control commands</li>
<li><code>rt/lowstate</code> - Publish robot state</li>
</ul>
<h2 id="running-the-system">Running the System</h2>
<h3 id="mock-mode-testing">Mock Mode (Testing)</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Terminal 1: Start RT thread</span>
./bin/ec_rt_thread

<span class="token comment"># Terminal 2: Start unitree bridge</span>
./bin/unitree_bridge

<span class="token comment"># Terminal 3: Run unitree_sdk2 example</span>
./g1_ankle_swing_example eth0
</code></pre>
<h3 id="real-hardware-mode">Real Hardware Mode</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Run with sudo for RT scheduling</span>
<span class="token function">sudo</span> ./bin/ec_rt_thread
./bin/unitree_bridge
</code></pre>
<h2 id="joint-mapping-reference">Joint Mapping Reference</h2>

<table>
<thead>
<tr>
<th>Joint ID</th>
<th>Name</th>
<th>Master</th>
<th>Body Part</th>
</tr>
</thead>
<tbody>
<tr>
<td>0-5</td>
<td>Left Leg</td>
<td>0</td>
<td>LeftHipPitch → LeftAnkleRoll</td>
</tr>
<tr>
<td>6-11</td>
<td>Right Leg</td>
<td>1</td>
<td>RightHipPitch → RightAnkleRoll</td>
</tr>
<tr>
<td>12-14</td>
<td>Waist</td>
<td>0</td>
<td>WaistYaw, WaistRoll, WaistPitch</td>
</tr>
<tr>
<td>15-21</td>
<td>Left Arm</td>
<td>2</td>
<td>LeftShoulderPitch → LeftWristYaw</td>
</tr>
<tr>
<td>22-28</td>
<td>Right Arm</td>
<td>3</td>
<td>RightShoulderPitch → RightWristYaw</td>
</tr>
</tbody>
</table>
