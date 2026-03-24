---


---

<h1 id="isaac-gr00t-n1.6-—-locomotion-architecture">Isaac GR00T N1.6 — Locomotion Architecture</h1>
<h2 id="what-gr00t-n1.6-is-and-isnt">What GR00T N1.6 Is (and Isn’t)</h2>
<p>GR00T N1.6 is a <strong>Vision-Language-Action (VLA) model</strong> focused on task-level reasoning and dexterous manipulation. It supports <em>loco-manipulation</em> (coordinated walking + grasping), but it does <strong>not</strong> directly output joint-level locomotion commands. It sits at the top of a hierarchical control stack.</p>
<h2 id="control-architecture">Control Architecture</h2>
<pre><code>GR00T N1.6 (VLA)
  - Language instruction understanding
  - Task decomposition and sequencing
  - Manipulation action output
        ↓
Whole-Body Controller (decoupled)
  ├── Lower body: RL-trained locomotion policy   ← separate, required
  └── Upper body: IK-based arm control
        ↓
Hardware / Simulation
</code></pre>
<p><strong>Yes — a separately trained RL policy is required for locomotion.</strong> GR00T N1.6 alone cannot make a humanoid walk.</p>
<h2 id="locomotion-resources-from-nvidia">Locomotion Resources from NVIDIA</h2>

<table>
<thead>
<tr>
<th>Resource</th>
<th>Purpose</th>
<th>Link</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>GR00T-WholeBodyControl</code></td>
<td>Decoupled WBC (RL legs + IK arms)</td>
<td><a href="https://github.com/NVlabs/GR00T-WholeBodyControl">https://github.com/NVlabs/GR00T-WholeBodyControl</a></td>
</tr>
<tr>
<td>GEAR-SONIC (Feb 2026)</td>
<td>Unified whole-body policy (single model)</td>
<td><a href="https://nvlabs.github.io/GEAR-SONIC/">https://nvlabs.github.io/GEAR-SONIC/</a></td>
</tr>
<tr>
<td>Isaac Lab</td>
<td>RL training environment</td>
<td><a href="https://isaac-sim.github.io/IsaacLab/">https://isaac-sim.github.io/IsaacLab/</a></td>
</tr>
<tr>
<td>COMPASS</td>
<td>Synthetic trajectory generation for navigation finetuning</td>
<td>NVIDIA internal</td>
</tr>
</tbody>
</table><h2 id="supported-robot-platforms-pre-trained-checkpoints">Supported Robot Platforms (Pre-Trained Checkpoints)</h2>
<ul>
<li>Unitree G1 (primary — most checkpoints)</li>
<li>Fourier GR-1</li>
<li>AgiBot Genie-1</li>
<li>Google Robot (Fractal dataset)</li>
<li>WidowX (Bridge dataset)</li>
<li>Franka Panda (LIBERO)</li>
<li>DROID robots</li>
</ul>
<p><strong>Topstar H2 is not a supported platform.</strong> No pre-trained GR00T or GEAR-SONIC checkpoint exists for it.</p>
<h2 id="gr00t-wholebodycontrol-vs.-gear-sonic">GR00T-WholeBodyControl vs. GEAR-SONIC</h2>

<table>
<thead>
<tr>
<th></th>
<th>GR00T-WBC (Decoupled)</th>
<th>GEAR-SONIC (Unified)</th>
</tr>
</thead>
<tbody>
<tr>
<td>Locomotion</td>
<td>RL policy (lower body)</td>
<td>Single policy (whole body)</td>
</tr>
<tr>
<td>Arm control</td>
<td>IK (upper body)</td>
<td>Same single policy</td>
</tr>
<tr>
<td>Coordination</td>
<td>Subsystems independent</td>
<td>Emergent whole-body coordination</td>
</tr>
<tr>
<td>Data needed</td>
<td>Locomotion RL + arm demos separately</td>
<td>100M+ mocap frames</td>
</tr>
<tr>
<td>Training effort</td>
<td>Lower</td>
<td>Much higher</td>
</tr>
<tr>
<td>Released</td>
<td>Yes</td>
<td>Yes (Feb 2026, Unitree G1 only)</td>
</tr>
</tbody>
</table><h2 id="key-takeaway">Key Takeaway</h2>
<p>To use GR00T N1.6 on a humanoid for loco-manipulation tasks:</p>
<ol>
<li>Train (or obtain) an RL locomotion policy for the target robot in Isaac Lab.</li>
<li>Deploy GR00T-WholeBodyControl as the mid-level WBC layer.</li>
<li>Run GR00T N1.6 VLA on top for language-conditioned task execution.</li>
</ol>
<h2 id="how-vla-and-rl-are-combined">How VLA and RL Are Combined</h2>
<p>It’s a <strong>hierarchical stack</strong> — VLA and RL are not fused into one model. They run at different<br>
frequencies and handle different parts of the body.</p>
<h3 id="the-architecture">The Architecture</h3>
<pre><code>  ┌──────────────────────────────────────────────┐
  │  GR00T N1.5/N1.6  (VLA)          ~10–30 Hz   │
  │  - Language instruction → task plan          │
  │  - Outputs: end-effector targets (6-DOF pose)│
  └──────────────────────┬───────────────────────┘
                         │ arm workspace targets
  ┌──────────────────────▼────────────────────────┐
  │  Whole-Body Controller (WBC)      ~50 Hz      │
  │  ┌─────────────────┐ ┌──────────────────────┐ │
  │  │  Upper body     │ │  Lower body          │ │
  │  │  IK solver      │ │  RL policy (PPO)     │ │
  │  │  (kinematic)    │ │  (your trained net)  │ │
  │  └────────┬────────┘ └───────────┬──────────┘ │
  └───────────┼──────────────────────┼────────────┘
              │  arm joint targets   │  leg joint targets
              └──────────┬───────────┘
  ┌──────────────────────▼───────────────────────┐
  │  Hardware / Simulation           1000 Hz     │
  │  Motor PD controllers                        │
  └──────────────────────────────────────────────┘
</code></pre>
<h3 id="what-each-layer-does">What each layer does</h3>
<h4 id="vla-gr00t-n1.6-—-the-brain">VLA (GR00T N1.6) — the “brain”</h4>
<ul>
<li>Takes camera images + language command as input</li>
<li>Reasons about the task: “pick up the cup and walk to the table”</li>
<li>Outputs arm end-effector pose targets — not joint angles, not torques</li>
<li>Runs slowly (~10–30 Hz); completely unaware of leg dynamics</li>
</ul>
<h4 id="rl-policy-—-the-legs">RL policy — the “legs”</h4>
<ul>
<li>Your standard PPO-trained locomotion policy (exactly what we’re training for H2)</li>
<li>Input: proprioception (joint pos/vel, IMU, velocity command)</li>
<li>Output: 12 leg joint position targets</li>
<li>Runs at 50 Hz, isolated from VLA</li>
<li>The velocity command it tracks can come from the VLA or from a joystick</li>
</ul>
<h4 id="ik-solver-—-the-arms-in-wbc-not-a-learned-model">IK solver — the “arms” (in WBC, not a learned model)</h4>
<ul>
<li>Converts VLA’s end-effector targets to arm joint angles</li>
<li>Runs at the same rate as the RL policy (~50 Hz)</li>
<li>Operates in the base frame — does not account for leg motion coupling</li>
</ul>
<h3 id="why-the-legs-and-arms-are-decoupled">Why the legs and arms are decoupled</h3>
<p>The WBC treats the torso as a fixed reference frame for IK. This works when locomotion is slow and<br>
smooth, but it means the arms don’t actively compensate for body sway. That’s the key limitation<br>
— and why unified policies like GEAR-SONIC or FALCON exist.</p>
<hr>
<h3 id="what-this-means-for-h2-practically">What this means for H2 practically</h3>
<p>The RL locomotion policy you’re training now is the lower-body component of this stack. Once it<br>
works, the path to loco-manipulation is:</p>
<p><strong>Step 1</strong> (now):    Train RL locomotion policy  ← current work<br>
<strong>Step 2</strong>:          Verify sim-to-real on H2 hardware<br>
<strong>Step 3</strong>:          Integrate GR00T-WBC: wrap the RL policy as the lower-body provider, add IK for arms via DDS<br>
<strong>Step 4</strong> (later):  Layer GR00T N1.6 VLA on top for language-conditioned tasks</p>
<p>The catch for H2 is that no GR00T pre-trained checkpoint exists for it. The VLA would need to be<br>
fine-tuned or a separate manipulation policy trained. The decoupled approach still makes sense for<br>
H2 precisely because it lets you solve each layer independently on a new platform.</p>

