---


---

<h2 id="lhandprolib--igh-ethercat-integration">LHandProLib + IgH EtherCAT Integration</h2>
<h3 id="background">Background</h3>
<p>The <strong>LHandProLib</strong> library provides a callback-based architecture that can work with any <em>EtherCAT</em> master:</p>
<ul>
<li><code>lhandprolib_set_send_rpdo_callback()</code> - register callback to send PDO data</li>
<li><code>lhandprolib_set_tpdo_data_decode()</code> - pass received PDO data for parsing</li>
</ul>
<p>This allows integrating with <strong>IgH</strong> <em>EtherCAT</em> master instead of the default <strong>SOEM</strong>.</p>
<h3 id="benefits">Benefits</h3>

<table>
<thead>
<tr>
<th>Aspect</th>
<th>Current <code>ls_hand.c</code></th>
<th>New <code>ls_hand_pro.c</code></th>
</tr>
</thead>
<tbody>
<tr>
<td>Protocol parsing</td>
<td>Manual</td>
<td>LHandProLib handles it</td>
</tr>
<tr>
<td>Sensor data</td>
<td>Basic</td>
<td>Rich API (normal, tangential, proximity)</td>
</tr>
<tr>
<td>Motor control</td>
<td>Raw PDO</td>
<td>High-level API (angles, velocity, homing)</td>
</tr>
<tr>
<td>Finger identification</td>
<td>Generic</td>
<td>Named (thumb, index, etc.)</td>
</tr>
</tbody>
</table><h3 id="proposed-changes">Proposed Changes</h3>
<h4 id="new-ls_hand_pro.c">[NEW] <code>ls_hand_pro.c</code></h4>
<p>New source file that combines:</p>
<ol>
<li><strong>IgH EtherCAT master</strong> - for real-time PDO exchange</li>
<li><strong>LHandProLib</strong> - for protocol parsing and high-level control</li>
</ol>
<p>Key integration points:</p>
<ul>
<li>Cyclic task calls lhandprolib_get_pre_send_rpdo_data() and writes to EtherCAT</li>
<li>Cyclic task reads EtherCAT and calls <code>lhandprolib_set_tpdo_data_decode()</code></li>
<li>RPDO callback bridges library to IgH master</li>
</ul>
<h4 id="modify-makefile">[MODIFY] <code>Makefile</code></h4>
<p>Add build target for ls_hand_pro with LHandProLib linking.</p>
<h4 id="verification-plan">Verification Plan</h4>
<ol>
<li>Build with make ls_hand_pro</li>
<li>Run and verify device enters OP state</li>
<li>Test enable/disable through LHandProLib API</li>
<li>Read sensor values using library functions</li>
</ol>

