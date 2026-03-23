---


---

<h1 id="sota-loco-manipulation-policies-—-2024–2026">SOTA Loco-Manipulation Policies — 2024–2026</h1>
<h2 id="overview-of-approaches">Overview of Approaches</h2>
<h3 id="unified-single-policy-systems">Unified Single-Policy Systems</h3>

<table>
<thead>
<tr>
<th>System</th>
<th>Venue / Date</th>
<th>Architecture</th>
<th>Training Data</th>
<th>Platform</th>
<th>Key Result</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>GEAR-SONIC</strong></td>
<td>NVIDIA, Feb 2026</td>
<td>42M-param transformer encoder-decoder</td>
<td>100M+ mocap frames (700h), 9000 GPU-h</td>
<td>Unitree G1</td>
<td>100% zero-shot success on 50 diverse trajectories</td>
</tr>
<tr>
<td><strong>HOVER</strong></td>
<td>ICRA 2025</td>
<td>1.5M-param transformer, mode-masked distillation</td>
<td>Simulation + proprioception</td>
<td>Humanoid</td>
<td>One policy for nav / loco-manip / tabletop; beats per-mode specialists</td>
</tr>
<tr>
<td><strong>WoCoCo</strong></td>
<td>2024</td>
<td>Contact-sequence RL, task-agnostic reward</td>
<td>Pure RL sim-to-real</td>
<td>Humanoid + dino robot</td>
<td>No motion priors; parkour, box manip, climbing</td>
</tr>
<tr>
<td><strong>ULC</strong></td>
<td>Jul 2025</td>
<td>Residual action + polynomial interpolation</td>
<td>RL</td>
<td>Unitree G1</td>
<td>Better tracking accuracy and workspace vs. decoupled baselines</td>
</tr>
<tr>
<td><strong>WholeBodyVLA</strong></td>
<td>ICLR 2026</td>
<td>LAM + dual-stream + RL loco-manip policy</td>
<td>Action-free egocentric video</td>
<td>Humanoid</td>
<td>Unified latent space for loco + manip; real-world deployment</td>
</tr>
<tr>
<td><strong>DreamControl</strong></td>
<td>2025</td>
<td>Diffusion transformer prior + RL</td>
<td>Human motion datasets</td>
<td>Unitree G1</td>
<td>Natural motions via diffusion prior; improved sim-to-real</td>
</tr>
<tr>
<td><strong>BeyondMimic</strong></td>
<td>Aug 2025</td>
<td>Diffusion + classifier guidance</td>
<td>Human mocap</td>
<td>Humanoid</td>
<td>Zero-shot task transfer via guidance; aerials, cartwheels</td>
</tr>
<tr>
<td><strong>FALCON</strong></td>
<td>L4DC 2026</td>
<td>Dual-agent RL (lower + upper body) + force curriculum</td>
<td>Simulation</td>
<td>G1 + Booster T1</td>
<td>2× tracking accuracy under 0–100 N loads; cross-platform</td>
</tr>
<tr>
<td><strong>BumbleBee</strong></td>
<td>2025</td>
<td>Cluster → expert policies → transformer distillation</td>
<td>Mocap + real-world delta</td>
<td>Humanoid</td>
<td>SOTA diversity + robustness; avoids data conflicts across motions</td>
</tr>
<tr>
<td><strong>HumanPlus</strong></td>
<td>CoRL 2024</td>
<td>Shadowing transformer (HST) + imitation (HIT)</td>
<td>40h human mocap + teleop</td>
<td>33-DOF custom</td>
<td>60–100% real-world success across 8 tasks</td>
</tr>
<tr>
<td><strong>OmniH2O</strong></td>
<td>CoRL 2024</td>
<td>Kinematic retargeting + DAgger distillation</td>
<td>RL → supervised</td>
<td>Humanoid</td>
<td>Universal VR/voice/RGB interface; sports + manipulation</td>
</tr>
<tr>
<td><strong>HARMON</strong></td>
<td>CoRL 2024</td>
<td>Human motion priors + VLM editing</td>
<td>Human motion datasets</td>
<td>Fourier GR-1</td>
<td>86.7% natural text-aligned motions (human eval)</td>
</tr>
<tr>
<td><strong>LeVERB</strong></td>
<td>2025</td>
<td>Hierarchical: VLA latent vocab + 1.1M transformer WBC</td>
<td>Synthetic only</td>
<td>Humanoid</td>
<td>80% visual nav; 7.8× better than naive hierarchical VLA</td>
</tr>
<tr>
<td><strong>SkillBlender</strong></td>
<td>Jun 2025</td>
<td>Pretrain primitive skills → dynamic blending</td>
<td>RL</td>
<td>3 humanoid morphologies</td>
<td>SkillBench: 8 loco-manip tasks, 3 embodiments</td>
</tr>
<tr>
<td><strong>MaskedManipulator</strong></td>
<td>SIGGRAPH Asia 2025</td>
<td>MimicManipulator tracker → MaskedManipulator distill</td>
<td>GRAB dataset (human-object)</td>
<td>Humanoid</td>
<td>Sparse goal spec + kinematic teleoperation from one unified policy</td>
</tr>
</tbody>
</table><h3 id="hierarchical--decoupled-baseline">Hierarchical / Decoupled (Baseline)</h3>

<table>
<thead>
<tr>
<th>System</th>
<th>Type</th>
<th>Architecture</th>
<th>Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>GR00T-WholeBodyControl</strong></td>
<td>Decoupled</td>
<td>RL (lower body) + IK (upper body)</td>
<td>Base layer under GR00T N1.5/N1.6 VLA; battle-tested, production-grade</td>
</tr>
<tr>
<td><strong>HOMIE</strong></td>
<td>Teleop + RL</td>
<td>Exoskeleton teleop + RL body policy</td>
<td>$500 exoskeleton; 2× faster than prior teleoperation systems</td>
</tr>
<tr>
<td><strong>HugWBC</strong></td>
<td>RL WBC</td>
<td>General command space + symmetrical loss</td>
<td>RSS 2025; versatile locomotion as base for upper-body tasks</td>
</tr>
</tbody>
</table><hr>
<h2 id="unified-single-policy-vs.-vla--rl-hierarchical">Unified Single Policy vs. VLA + RL (Hierarchical)</h2>

<table>
<thead>
<tr>
<th>Dimension</th>
<th>Unified Single Policy</th>
<th>VLA (Manipulation) + RL (Locomotion)</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Whole-body coordination</strong></td>
<td>Emergent natural coupling between legs and arms</td>
<td>Arms and legs are independent subsystems</td>
</tr>
<tr>
<td><strong>Manipulation under motion</strong></td>
<td>Handles dynamic coupling (reach while walking, resist forces)</td>
<td>IK operates in fixed frame; degrades under base perturbation</td>
</tr>
<tr>
<td><strong>Manipulation workspace</strong></td>
<td>Larger — trunk and legs compensate naturally</td>
<td>Limited by IK reachability from nominal stance</td>
</tr>
<tr>
<td><strong>Training complexity</strong></td>
<td>High — requires curriculum, large data, careful sim-to-real</td>
<td>Lower — solve legs and arms independently</td>
</tr>
<tr>
<td><strong>Data requirements</strong></td>
<td>High — diverse whole-body demos or large-scale mocap</td>
<td>Low — locomotion RL and arm demos are separate, well-understood</td>
</tr>
<tr>
<td><strong>Sim-to-real transfer</strong></td>
<td>Harder — full-body contact dynamics are complex</td>
<td>More reliable — RL locomotion for legs is a mature pipeline</td>
</tr>
<tr>
<td><strong>New robot adaptation</strong></td>
<td>Full retraining required</td>
<td>Swap only the leg RL policy; IK arm control transfers easily</td>
</tr>
<tr>
<td><strong>Language / task generalisation</strong></td>
<td>Requires paired VLA (e.g., WholeBodyVLA); achievable</td>
<td>VLA sits on top; locomotion subsystem unchanged</td>
</tr>
<tr>
<td><strong>Force handling</strong></td>
<td>Learned implicitly (FALCON, ULC show strong results)</td>
<td>IK is kinematic-only; force awareness requires extra layers</td>
</tr>
<tr>
<td><strong>Runtime compute</strong></td>
<td>Single inference pass (efficient)</td>
<td>Two inference calls + IK solve (more overhead)</td>
</tr>
<tr>
<td><strong>Maturity (2026)</strong></td>
<td>Advancing fast; competitive with hierarchical on benchmarks</td>
<td>Battle-tested; dominant in deployed production systems</td>
</tr>
</tbody>
</table><hr>
<h2 id="decision-guide">Decision Guide</h2>
<h3 id="choose-unified-policy-if">Choose Unified Policy if:</h3>
<ul>
<li>You need tight whole-body coordination (payload carry, door opening while walking, dynamic reaching)</li>
<li>You have access to large-scale mocap or simulation data for training</li>
<li>You are starting from scratch on a well-supported platform (e.g., Unitree G1)</li>
<li>You want a single model to maintain and deploy</li>
</ul>
<h3 id="choose-vla--rl-hierarchical-if">Choose VLA + RL (Hierarchical) if:</h3>
<ul>
<li>You already have a working locomotion policy and want to add task-level intelligence cheaply</li>
<li>Your manipulation tasks are relatively independent of locomotion (tabletop-style while walking slowly)</li>
<li>You want faster iteration — fix locomotion and manipulation bugs in isolation</li>
<li>You are on a new/unsupported robot platform where training data is scarce</li>
</ul>
<hr>
<h2 id="practical-notes-for-topstar-h2">Practical Notes for Topstar H2</h2>
<p>The H2 is not supported by any of the SOTA unified checkpoints above (all primarily target Unitree G1 or Fourier GR-1). The most practical path:</p>
<ol>
<li><strong>Short term:</strong> CPG gait (<code>h2_gait.c</code>) + balance controller (<code>h2_balance.cpp</code>) — no training needed.</li>
<li><strong>Medium term:</strong> Train an RL locomotion policy in Isaac Lab using <code>model/urdf/h2.xml</code>, export to ONNX, deploy via <code>h2_rl_runner.py</code>.</li>
<li><strong>Long term:</strong> Once RL locomotion is stable, layer a manipulation policy or VLA on top via DDS, or pursue a unified policy trained on H2-specific data.</li>
</ol>
<p>The HOVER / ULC architecture (transformer-based distillation from multiple experts) is a strong candidate for the eventual unified policy, as it requires less raw mocap data than GEAR-SONIC while still outperforming decoupled methods.</p>
<hr>
<h2 id="references">References</h2>

<table>
<thead>
<tr>
<th>Paper</th>
<th>arXiv / Link</th>
</tr>
</thead>
<tbody>
<tr>
<td>GEAR-SONIC</td>
<td><a href="https://nvlabs.github.io/GEAR-SONIC/">https://nvlabs.github.io/GEAR-SONIC/</a></td>
</tr>
<tr>
<td>HOVER</td>
<td><a href="https://arxiv.org/abs/2410.21229">https://arxiv.org/abs/2410.21229</a></td>
</tr>
<tr>
<td>WoCoCo</td>
<td><a href="https://arxiv.org/abs/2406.06005">https://arxiv.org/abs/2406.06005</a></td>
</tr>
<tr>
<td>ULC</td>
<td><a href="https://arxiv.org/abs/2507.06905">https://arxiv.org/abs/2507.06905</a></td>
</tr>
<tr>
<td>WholeBodyVLA</td>
<td><a href="https://github.com/OpenDriveLab/WholebodyVLA">https://github.com/OpenDriveLab/WholebodyVLA</a></td>
</tr>
<tr>
<td>DreamControl</td>
<td><a href="https://arxiv.org/abs/2509.14353">https://arxiv.org/abs/2509.14353</a></td>
</tr>
<tr>
<td>BeyondMimic</td>
<td><a href="https://arxiv.org/abs/2508.08241">https://arxiv.org/abs/2508.08241</a></td>
</tr>
<tr>
<td>FALCON</td>
<td><a href="https://arxiv.org/abs/2505.06776">https://arxiv.org/abs/2505.06776</a></td>
</tr>
<tr>
<td>BumbleBee</td>
<td><a href="https://arxiv.org/abs/2506.12779">https://arxiv.org/abs/2506.12779</a></td>
</tr>
<tr>
<td>HumanPlus</td>
<td><a href="https://arxiv.org/abs/2406.10454">https://arxiv.org/abs/2406.10454</a></td>
</tr>
<tr>
<td>OmniH2O</td>
<td><a href="https://arxiv.org/abs/2406.08858">https://arxiv.org/abs/2406.08858</a></td>
</tr>
<tr>
<td>HARMON</td>
<td><a href="https://arxiv.org/abs/2410.12773">https://arxiv.org/abs/2410.12773</a></td>
</tr>
<tr>
<td>LeVERB</td>
<td><a href="https://arxiv.org/abs/2506.13751">https://arxiv.org/abs/2506.13751</a></td>
</tr>
<tr>
<td>SkillBlender</td>
<td><a href="https://arxiv.org/abs/2506.09366">https://arxiv.org/abs/2506.09366</a></td>
</tr>
<tr>
<td>MaskedManipulator</td>
<td><a href="https://arxiv.org/abs/2505.19086">https://arxiv.org/abs/2505.19086</a></td>
</tr>
<tr>
<td>HiLo</td>
<td><a href="https://arxiv.org/abs/2502.03122">https://arxiv.org/abs/2502.03122</a></td>
</tr>
<tr>
<td>HugWBC</td>
<td><a href="https://arxiv.org/abs/2502.03206">https://arxiv.org/abs/2502.03206</a></td>
</tr>
<tr>
<td>HOMIE</td>
<td><a href="https://arxiv.org/abs/2502.13013">https://arxiv.org/abs/2502.13013</a></td>
</tr>
<tr>
<td>ULTRA</td>
<td><a href="https://arxiv.org/abs/2603.03279">https://arxiv.org/abs/2603.03279</a></td>
</tr>
<tr>
<td>GR00T-WholeBodyControl</td>
<td><a href="https://github.com/NVlabs/GR00T-WholeBodyControl">https://github.com/NVlabs/GR00T-WholeBodyControl</a></td>
</tr>
</tbody>
</table>
