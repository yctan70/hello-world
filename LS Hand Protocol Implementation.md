---


---

<h1 id="ls-hand-protocol-implementation---summary">LS Hand Protocol Implementation - Summary</h1>
<h2 id="changes-made">Changes Made</h2>
<h3 id="ls_hand.h"><code>ls_hand.h</code></h3>
<h4 id="added">Added:</h4>
<ul>
<li>Protocol constants (frame types, PDO indices)</li>
<li><code>LSHand_AxisMode</code>  enum (position, velocity, torque, hybrid modes)</li>
<li><code>LSHand_AxisControl</code>  enum (enable, disable, home, e-stop, etc.)</li>
<li>Axis status bit definitions</li>
<li><code>LSHand_AxisCmd</code>,  <code>LSHand_AxisFeedback</code>,  <code>LSHand_Status</code>  structures</li>
<li>Control API declarations</li>
</ul>
<h3 id="ls_hand.c"><code>ls_hand.c</code></h3>
<h4 id="added-1">Added:</h4>
<ul>
<li><code>axis_commands[]</code>  storage for per-axis commands</li>
<li><code>build_rxpdo_frame()</code>  - builds command <em>PDO</em> from axis commands</li>
<li><code>LS_hand_enable()</code> /<code>LS_hand_disable()</code>  - enable/disable all axes</li>
<li><code>LS_hand_set_mode()</code>/<code>LS_hand_set_control()</code>  - per-axis mode/control</li>
<li><code>LS_hand_set_position()</code> / <code>LS_hand_set_velocity()</code> / <code>LS_hand_set_current()</code>  - motion commands</li>
<li><code>LS_hand_get_status()</code> /<code>LS_hand_is_enabled()</code>  - feedback functions</li>
</ul>
<h4 id="fixed">Fixed:</h4>
<ul>
<li>PDO subindex off-by-one error (subindexes start from 1, not 0)</li>
<li><code>EC_READ_S16</code>/<code>EC_WRITE_S16</code>  macro usage</li>
<li>Added missing  <code>DEFAULT_PRIORITY</code>  definition</li>
<li><code>run</code>  flag reset before thread start</li>
</ul>
<h4 id="protocol-summary-from-dh116_protocol.xlsx">Protocol Summary (from <code>DH116_Protocol.xlsx</code>)</h4>

<table>
<thead>
<tr>
<th>Control</th>
<th>Command</th>
<th>Value</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>IDLE</td>
<td>0</td>
<td>No command</td>
<td></td>
</tr>
<tr>
<td>ENABLE</td>
<td>1</td>
<td>Enable axis</td>
<td></td>
</tr>
<tr>
<td>DISABLE</td>
<td>2</td>
<td>Disable axis</td>
<td></td>
</tr>
<tr>
<td>HOME</td>
<td>4</td>
<td>Trigger homing</td>
<td></td>
</tr>
<tr>
<td>E-STOP</td>
<td>7</td>
<td>Emergency stop</td>
<td></td>
</tr>
</tbody>
</table><h2 id="build-status">Build Status</h2>
<p>✅ Compiles successfully with minor warnings about  <code>CPU_ZERO</code>/<code>CPU_SET</code>  macros (need  <code>_GNU_SOURCE</code>).</p>
<h2 id="usage-example">Usage Example</h2>
<pre class=" language-c"><code class="prism  language-c"><span class="token comment">// Activate EtherCAT connection</span>
<span class="token function">LS_hand_activate</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// Enable all axes</span>
<span class="token function">LS_hand_enable</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token comment">// Set position for axis 0</span>
<span class="token function">LS_hand_set_position</span><span class="token punctuation">(</span><span class="token number">0</span><span class="token punctuation">,</span> <span class="token number">5000</span><span class="token punctuation">)</span><span class="token punctuation">;</span> <span class="token comment">// 50% travel</span>

<span class="token comment">// Check status</span>
LSHand_Status status<span class="token punctuation">;</span>
<span class="token function">LS_hand_get_status</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>status<span class="token punctuation">)</span><span class="token punctuation">;</span>

<span class="token keyword">if</span> <span class="token punctuation">(</span>status<span class="token punctuation">.</span>axes<span class="token punctuation">[</span><span class="token number">0</span><span class="token punctuation">]</span><span class="token punctuation">.</span>status <span class="token operator">&amp;</span> AXIS_STATUS_ENABLED<span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token function">printf</span><span class="token punctuation">(</span><span class="token string">"Axis 0 enabled\n"</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span>

<span class="token comment">// Disable and shutdown</span>
<span class="token function">LS_hand_disable</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">LS_hand_deactivate</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<h3 id="adding-sensor-data-mapping">Adding Sensor Data Mapping</h3>
<p>Added sensor structures and API functions. Now expanding the parsed sensor structure to include tangential force, direction, proximity, and palm normal force.</p>
<h3 id="updated-lshand_fingersensor-structure-30-bytes-total">Updated <code>LSHand_FingerSensor</code> structure (30 bytes total):</h3>

<table>
<thead>
<tr>
<th>Bytes</th>
<th>Field</th>
<th>Type</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>0-8</td>
<td>channel[9]</td>
<td>uint8_t[]</td>
<td>9 capacitive electrode channels (0-255)</td>
</tr>
<tr>
<td>9-10</td>
<td>normal_force</td>
<td>uint16_t</td>
<td>Normal force (0.01N resolution)</td>
</tr>
<tr>
<td>11-12</td>
<td>tangential_x</td>
<td>int16_t</td>
<td>Tangential force X component</td>
</tr>
<tr>
<td>13-14</td>
<td>tangential_y</td>
<td>int16_t</td>
<td>Tangential force Y component</td>
</tr>
<tr>
<td>15-16</td>
<td>direction</td>
<td>uint16_t</td>
<td>Direction (0-36000 = 0-360.00°)</td>
</tr>
<tr>
<td>17-18</td>
<td>proximity</td>
<td>uint16_t</td>
<td>Proximity/distance value</td>
</tr>
<tr>
<td>19-20</td>
<td>palm_force</td>
<td>uint16_t</td>
<td>Palm normal force</td>
</tr>
<tr>
<td>21-29</td>
<td>reserved[9]</td>
<td>uint8_t[]</td>
<td>Reserved for future</td>
</tr>
</tbody>
</table>
