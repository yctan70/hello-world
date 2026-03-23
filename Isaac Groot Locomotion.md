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

