---


---

<h2 id="ps2-controller-integration">PS2 Controller Integration</h2>
<h3 id="summary">Summary</h3>
<p>Successfully integrated a Bluetooth PS2 controller (connected at <code>/dev/input/js0</code>) with the unitree_sdk2 framework. The controller data is now packed into the 40-byte <code>wireless_remote</code>  field in the  <code>rt/lowstate</code>  message.</p>
<h3 id="files-created">Files Created</h3>
<h4 id="ps2_joystick.h"><code>ps2_joystick.h</code></h4>
<p>Header file defining:</p>
<ul>
<li><code>PS2JoystickState</code>  structure for storing controller state</li>
<li><code>WirelessRemoteData</code>  packed structure (40 bytes) for unitree_sdk2 format</li>
<li>Key bitmask constants (<code>PS2_KEY_A</code>,  <code>PS2_KEY_B</code>,  <code>PS2_KEY_L1</code>, etc.)</li>
<li>API functions for joystick operations</li>
</ul>
<h4 id="ps2_joystick.c"><code>ps2_joystick.c</code></h4>
<p>Implementation using Linux joystick API:</p>
<ul>
<li>Non-blocking read from <code>/dev/input/js0</code></li>
<li>Maps PS2 axes (lx, ly, rx, ry) from int16 to float (-1.0 to 1.0)</li>
<li>Maps buttons to 16-bit bitmask matching unitree format</li>
<li>Packs data into 40-byte</li>
</ul>
<h4 id="test_ps2_controller.c"><code>test_ps2_controller.c</code></h4>
<p>Standalone test program with visual feedback:</p>
<ul>
<li>ASCII art joystick position display</li>
<li>Button state indicators</li>
<li>Hex dump of packed  <code>wireless_remote[40]</code>  data</li>
</ul>
<h3 id="modified-files">Modified files</h3>
<ul>
<li><code>src/ec_shared_mem.h</code>  - Added  <code>JoystickData</code>  structure</li>
<li><code>src/unitree_bridge.cpp</code>  - Integrated joystick reading and packing into  <code>wireless_remote[40]</code></li>
<li><code>src/CMakeLists.txt</code>  - Added build target</li>
</ul>
<h3 id="fixing-ps2-button-mapping">Fixing PS2 Button Mapping</h3>
<p>Updated the button mapping in <code>ps2_joystick.h</code> to match your controller:</p>

<table>
<thead>
<tr>
<th>Button</th>
<th>Index</th>
</tr>
</thead>
<tbody>
<tr>
<td>X (Square)</td>
<td>3</td>
</tr>
<tr>
<td>Y (Triangle)</td>
<td>4</td>
</tr>
<tr>
<td>L1</td>
<td>6</td>
</tr>
<tr>
<td>R1</td>
<td>7</td>
</tr>
<tr>
<td>L2</td>
<td>8 (AXIS 5)</td>
</tr>
<tr>
<td>R2</td>
<td>9 (AXIS 4)</td>
</tr>
<tr>
<td>Select</td>
<td>10</td>
</tr>
<tr>
<td>Start</td>
<td>11</td>
</tr>
<tr>
<td>D-pad</td>
<td>AXIS 6/7</td>
</tr>
</tbody>
</table><h3 id="fixing-wireless-remote-format">Fixing Wireless Remote Format</h3>
<p>Fixed the <code>wireless_remote</code> data format to match the <em>unitree</em> <code>xRockerBtnDataStruct</code> structure. The issue was that I had the wrong byte layout.</p>
<p><strong>Correct format (40 bytes)</strong>:</p>

<table>
<thead>
<tr>
<th>Bytes</th>
<th>Field</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>0-1</td>
<td>head[2]</td>
<td>Header (zeros)</td>
</tr>
<tr>
<td>2-3</td>
<td>keys</td>
<td>Button bitmask (uint16)</td>
</tr>
<tr>
<td>4-7</td>
<td>lx</td>
<td>Left stick X (float)</td>
</tr>
<tr>
<td>8-11</td>
<td>rx</td>
<td>Right stick X (float)</td>
</tr>
<tr>
<td>12-15</td>
<td>ry</td>
<td>Right stick Y (float)</td>
</tr>
<tr>
<td>16-19</td>
<td>L2</td>
<td>L2 trigger (float)</td>
</tr>
<tr>
<td>20-23</td>
<td>ly</td>
<td>Left stick Y (float)</td>
</tr>
<tr>
<td>24-39</td>
<td>idle[16]</td>
<td>Padding</td>
</tr>
</tbody>
</table>
