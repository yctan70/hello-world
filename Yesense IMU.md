---


---

<h1 id="yesense-imu-integration---walkthrough">Yesense IMU Integration - Walkthrough</h1>
<p>This document describes the integration of the Yesense IMU sensor with the G1 Robot motor control system, enabling IMU data to be published via the <code>unitree_sdk2</code>-compatible DDS bridge.</p>
<h2 id="overview">Overview</h2>
<p>The integration consists of three main components:</p>
<ol>
<li><strong><code>imu_serial.c/h</code></strong> - Low-level serial driver that reads and parses Yesense protocol data</li>
<li><strong>Shared Memory</strong> - <code>IMUData</code> structure in <code>ec_shared_mem.h</code> for inter-process communication</li>
<li><strong><code>unitree_bridge.cpp</code></strong> - Reads IMU data, stores in shared memory, and publishes via DDS</li>
</ol>
<pre class=" language-mermaid"><svg id="mermaid-svg-foHys5MdAchU7jMo" width="100%" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" height="89.4375" style="max-width: 1460.703125px;" viewBox="0 0 1460.703125 89.4375"><style>#mermaid-svg-foHys5MdAchU7jMo{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#000000;}#mermaid-svg-foHys5MdAchU7jMo .error-icon{fill:#552222;}#mermaid-svg-foHys5MdAchU7jMo .error-text{fill:#552222;stroke:#552222;}#mermaid-svg-foHys5MdAchU7jMo .edge-thickness-normal{stroke-width:2px;}#mermaid-svg-foHys5MdAchU7jMo .edge-thickness-thick{stroke-width:3.5px;}#mermaid-svg-foHys5MdAchU7jMo .edge-pattern-solid{stroke-dasharray:0;}#mermaid-svg-foHys5MdAchU7jMo .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-svg-foHys5MdAchU7jMo .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-svg-foHys5MdAchU7jMo .marker{fill:#666;stroke:#666;}#mermaid-svg-foHys5MdAchU7jMo .marker.cross{stroke:#666;}#mermaid-svg-foHys5MdAchU7jMo svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-svg-foHys5MdAchU7jMo .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#000000;}#mermaid-svg-foHys5MdAchU7jMo .cluster-label text{fill:#333;}#mermaid-svg-foHys5MdAchU7jMo .cluster-label span{color:#333;}#mermaid-svg-foHys5MdAchU7jMo .label text,#mermaid-svg-foHys5MdAchU7jMo span{fill:#000000;color:#000000;}#mermaid-svg-foHys5MdAchU7jMo .node rect,#mermaid-svg-foHys5MdAchU7jMo .node circle,#mermaid-svg-foHys5MdAchU7jMo .node ellipse,#mermaid-svg-foHys5MdAchU7jMo .node polygon,#mermaid-svg-foHys5MdAchU7jMo .node path{fill:#eee;stroke:#999;stroke-width:1px;}#mermaid-svg-foHys5MdAchU7jMo .node .label{text-align:center;}#mermaid-svg-foHys5MdAchU7jMo .node.clickable{cursor:pointer;}#mermaid-svg-foHys5MdAchU7jMo .arrowheadPath{fill:#333333;}#mermaid-svg-foHys5MdAchU7jMo .edgePath .path{stroke:#666;stroke-width:1.5px;}#mermaid-svg-foHys5MdAchU7jMo .flowchart-link{stroke:#666;fill:none;}#mermaid-svg-foHys5MdAchU7jMo .edgeLabel{background-color:white;text-align:center;}#mermaid-svg-foHys5MdAchU7jMo .edgeLabel rect{opacity:0.5;background-color:white;fill:white;}#mermaid-svg-foHys5MdAchU7jMo .cluster rect{fill:hsl(210,66.6666666667%,95%);stroke:#26a;stroke-width:1px;}#mermaid-svg-foHys5MdAchU7jMo .cluster text{fill:#333;}#mermaid-svg-foHys5MdAchU7jMo .cluster span{color:#333;}#mermaid-svg-foHys5MdAchU7jMo div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(-160,0%,93.3333333333%);border:1px solid #26a;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-svg-foHys5MdAchU7jMo:root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}#mermaid-svg-foHys5MdAchU7jMo flowchart{fill:apa;}</style><g><g class="output"><g class="clusters"></g><g class="edgePaths"><g class="edgePath LS-A LE-B" id="L-A-B" style="opacity: 1;"><path class="path" d="M122.578125,44.71875L216.96875,44.71875L311.359375,44.71875" marker-end="url(https://stackedit.io/app#arrowhead11)" style="fill:none"></path><defs><marker id="arrowhead11" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath LS-B LE-C" id="L-B-C" style="opacity: 1;"><path class="path" d="M416.71875,44.71875L494.1875,44.71875L571.65625,44.71875" marker-end="url(https://stackedit.io/app#arrowhead12)" style="fill:none"></path><defs><marker id="arrowhead12" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath LS-C LE-D" id="L-C-D" style="opacity: 1;"><path class="path" d="M680.59375,44.71875L724.2734375,44.71875L767.953125,44.71875" marker-end="url(https://stackedit.io/app#arrowhead13)" style="fill:none"></path><defs><marker id="arrowhead13" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath LS-D LE-E" id="L-D-E" style="opacity: 1;"><path class="path" d="M901.78125,44.71875L945.90625,44.71875L990.03125,44.71875" marker-end="url(https://stackedit.io/app#arrowhead14)" style="fill:none"></path><defs><marker id="arrowhead14" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g><g class="edgePath LS-E LE-F" id="L-E-F" style="opacity: 1;"><path class="path" d="M1142.578125,44.71875L1212.9296875,44.71875L1283.28125,44.71875" marker-end="url(https://stackedit.io/app#arrowhead15)" style="fill:none"></path><defs><marker id="arrowhead15" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="strokeWidth" markerWidth="8" markerHeight="6" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowheadPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker></defs></g></g><g class="edgeLabels"><g class="edgeLabel" transform="translate(216.96875,44.71875)" style="opacity: 1;"><g transform="translate(-69.390625,-13.359375)" class="label"><rect rx="0" ry="0" width="138.78125" height="26.71875"></rect><foreignObject width="138.78125" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-A-B" class="edgeLabel L-LS-A' L-LE-B">Serial 460800 baud</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(494.1875,44.71875)" style="opacity: 1;"><g transform="translate(-52.46875,-13.359375)" class="label"><rect rx="0" ry="0" width="104.9375" height="26.71875"></rect><foreignObject width="104.9375" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-B-C" class="edgeLabel L-LS-B' L-LE-C">Parse Protocol</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(724.2734375,44.71875)" style="opacity: 1;"><g transform="translate(-18.6796875,-13.359375)" class="label"><rect rx="0" ry="0" width="37.359375" height="26.71875"></rect><foreignObject width="37.359375" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-C-D" class="edgeLabel L-LS-C' L-LE-D">Copy</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(945.90625,44.71875)" style="opacity: 1;"><g transform="translate(-19.125,-13.359375)" class="label"><rect rx="0" ry="0" width="38.25" height="26.71875"></rect><foreignObject width="38.25" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-D-E" class="edgeLabel L-LS-D' L-LE-E">Read</span></div></foreignObject></g></g><g class="edgeLabel" transform="translate(1212.9296875,44.71875)" style="opacity: 1;"><g transform="translate(-45.3515625,-13.359375)" class="label"><rect rx="0" ry="0" width="90.703125" height="26.71875"></rect><foreignObject width="90.703125" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span id="L-L-E-F" class="edgeLabel L-LS-E' L-LE-F">DDS Publish</span></div></foreignObject></g></g></g><g class="nodes"><g class="node default" id="flowchart-A-224" transform="translate(65.2890625,44.71875)" style="opacity: 1;"><rect rx="0" ry="0" x="-57.2890625" y="-36.71875" width="114.578125" height="73.4375" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-47.2890625,-26.71875)"><foreignObject width="94.578125" height="53.4375"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Yesense IMU<br>/dev/ttyUSB0</div></foreignObject></g></g></g><g class="node default" id="flowchart-B-225" transform="translate(364.0390625,44.71875)" style="opacity: 1;"><rect rx="0" ry="0" x="-52.6796875" y="-23.359375" width="105.359375" height="46.71875" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-42.6796875,-13.359375)"><foreignObject width="85.359375" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">imu_serial.c</div></foreignObject></g></g></g><g class="node default" id="flowchart-C-227" transform="translate(626.125,44.71875)" style="opacity: 1;"><rect rx="0" ry="0" x="-54.46875" y="-23.359375" width="108.9375" height="46.71875" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-44.46875,-13.359375)"><foreignObject width="88.9375" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">IMUReading</div></foreignObject></g></g></g><g class="node default" id="flowchart-D-229" transform="translate(834.8671875,44.71875)" style="opacity: 1;"><rect rx="0" ry="0" x="-66.9140625" y="-36.71875" width="133.828125" height="73.4375" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-56.9140625,-26.71875)"><foreignObject width="113.828125" height="53.4375"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">Shared Memory<br>IMUData</div></foreignObject></g></g></g><g class="node default" id="flowchart-E-231" transform="translate(1066.3046875,44.71875)" style="opacity: 1;"><rect rx="0" ry="0" x="-76.2734375" y="-23.359375" width="152.546875" height="46.71875" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-66.2734375,-13.359375)"><foreignObject width="132.546875" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">unitree_bridge.cpp</div></foreignObject></g></g></g><g class="node default" id="flowchart-F-233" transform="translate(1367.9921875,44.71875)" style="opacity: 1;"><rect rx="0" ry="0" x="-84.7109375" y="-36.71875" width="169.421875" height="73.4375" class="label-container"></rect><g class="label" transform="translate(0,0)"><g transform="translate(-74.7109375,-26.71875)"><foreignObject width="149.421875" height="53.4375"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;">rt/lowstate<br>LowState_.imu_state</div></foreignObject></g></g></g></g></g></g></svg></pre>
<hr>
<h2 id="component-details">Component Details</h2>
<h3 id="imu-serial-driver-imu_serial.ch">1. IMU Serial Driver (<code>imu_serial.c/h</code>)</h3>
<p><strong>Location:</strong> [imu_serial.h](file:///home/yctan/g1_motor_control/src/imu_serial.h) | [imu_serial.c](file:///home/yctan/g1_motor_control/src/imu_serial.c)</p>
<p><strong>Purpose:</strong> Reads raw bytes from the Yesense IMU serial port, parses the TLV-based protocol, and outputs sensor data in a normalized format.</p>
<h4 id="key-features">Key Features</h4>

<table>
<thead>
<tr>
<th>Feature</th>
<th>Implementation</th>
</tr>
</thead>
<tbody>
<tr>
<td>Baud Rate</td>
<td>460800 (configurable)</td>
</tr>
<tr>
<td>Protocol</td>
<td>Yesense binary TLV format</td>
</tr>
<tr>
<td>Buffering</td>
<td>Ring buffer for async reads</td>
</tr>
<tr>
<td>CRC Validation</td>
<td>2-byte checksum verification</td>
</tr>
<tr>
<td>Unit Conversion</td>
<td>deg→rad for angles, deg/s→rad/s for gyro</td>
</tr>
</tbody>
</table><h4 id="data-output-structure">Data Output Structure</h4>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">typedef</span> <span class="token keyword">struct</span> <span class="token punctuation">{</span>
  <span class="token keyword">float</span> quaternion<span class="token punctuation">[</span><span class="token number">4</span><span class="token punctuation">]</span><span class="token punctuation">;</span>    <span class="token comment">// w, x, y, z</span>
  <span class="token keyword">float</span> gyroscope<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">;</span>     <span class="token comment">// rad/s</span>
  <span class="token keyword">float</span> accelerometer<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">;</span> <span class="token comment">// m/s²</span>
  <span class="token keyword">float</span> rpy<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">;</span>           <span class="token comment">// rad (roll, pitch, yaw)</span>
  int16_t temperature<span class="token punctuation">;</span>    <span class="token comment">// Celsius * 100</span>
  uint64_t timestamp_us<span class="token punctuation">;</span>  <span class="token comment">// Sample timestamp</span>
  bool valid<span class="token punctuation">;</span>
<span class="token punctuation">}</span> IMUReading<span class="token punctuation">;</span>
</code></pre>
<h4 id="yesense-protocol-parsing">Yesense Protocol Parsing</h4>
<p>The driver parses these TLV data IDs:</p>

<table>
<thead>
<tr>
<th>Data ID</th>
<th>Name</th>
<th>Length</th>
<th>Conversion</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>0x01</code></td>
<td>Temperature</td>
<td>2 bytes</td>
<td>× 0.01</td>
</tr>
<tr>
<td><code>0x10</code></td>
<td>Accelerometer</td>
<td>12 bytes</td>
<td>× 1e-6 m/s²</td>
</tr>
<tr>
<td><code>0x20</code></td>
<td>Gyroscope</td>
<td>12 bytes</td>
<td>× 1e-6 deg/s → rad/s</td>
</tr>
<tr>
<td><code>0x40</code></td>
<td>Euler angles</td>
<td>12 bytes</td>
<td>× 1e-6 deg → rad</td>
</tr>
<tr>
<td><code>0x41</code></td>
<td>Quaternion</td>
<td>16 bytes</td>
<td>× 1e-6</td>
</tr>
<tr>
<td><code>0x51</code></td>
<td>Timestamp</td>
<td>4 bytes</td>
<td>microseconds</td>
</tr>
</tbody>
</table><hr>
<h3 id="shared-memory-structure-ec_shared_mem.h">2. Shared Memory Structure (<code>ec_shared_mem.h</code>)</h3>
<p><strong>Location:</strong> [ec_shared_mem.h](file:///home/yctan/g1_motor_control/src/ec_shared_mem.h)</p>
<p>The IMU data is stored in the <code>SharedMemoryData</code> structure alongside motor data:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">typedef</span> <span class="token keyword">struct</span> <span class="token punctuation">{</span>
  <span class="token keyword">float</span> quaternion<span class="token punctuation">[</span><span class="token number">4</span><span class="token punctuation">]</span><span class="token punctuation">;</span>    <span class="token comment">// Orientation (w,x,y,z)</span>
  <span class="token keyword">float</span> gyroscope<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">;</span>     <span class="token comment">// Angular velocity (rad/s)</span>
  <span class="token keyword">float</span> accelerometer<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">;</span> <span class="token comment">// Linear acceleration (m/s²)</span>
  <span class="token keyword">float</span> rpy<span class="token punctuation">[</span><span class="token number">3</span><span class="token punctuation">]</span><span class="token punctuation">;</span>           <span class="token comment">// Roll, Pitch, Yaw (rad)</span>
  int16_t temperature<span class="token punctuation">;</span>    <span class="token comment">// IMU temperature</span>
  uint8_t padding<span class="token punctuation">[</span><span class="token number">2</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
  uint64_t timestamp_ns<span class="token punctuation">;</span>  <span class="token comment">// Sample timestamp</span>
  bool valid<span class="token punctuation">;</span>             <span class="token comment">// Data valid flag</span>
  uint8_t padding2<span class="token punctuation">[</span><span class="token number">7</span><span class="token punctuation">]</span><span class="token punctuation">;</span>
<span class="token punctuation">}</span> IMUData<span class="token punctuation">;</span>
</code></pre>
<p>This structure is compatible with <code>unitree_sdk2</code>'s <code>IMUState_</code> format.</p>
<hr>
<h3 id="unitree-bridge-integration-unitree_bridge.cpp">3. Unitree Bridge Integration (<code>unitree_bridge.cpp</code>)</h3>
<p><strong>Location:</strong> [unitree_bridge.cpp](file:///home/yctan/g1_motor_control/src/unitree_bridge.cpp)</p>
<h4 id="initialization">Initialization</h4>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token comment">// Initialize IMU serial port</span>
imu_fd <span class="token operator">=</span> <span class="token function">imu_serial_init</span><span class="token punctuation">(</span>IMU_DEFAULT_DEVICE<span class="token punctuation">,</span> IMU_DEFAULT_BAUDRATE<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token keyword">if</span> <span class="token punctuation">(</span>imu_fd <span class="token operator">&lt;</span> <span class="token number">0</span><span class="token punctuation">)</span> <span class="token punctuation">{</span>
  std<span class="token operator">::</span>cerr <span class="token operator">&lt;&lt;</span> <span class="token string">"Warning: IMU not available, continuing without IMU"</span> <span class="token operator">&lt;&lt;</span> std<span class="token operator">::</span>endl<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<blockquote>
<p>[!NOTE]<br>
The bridge continues to operate even if the IMU is not connected, providing default values (identity quaternion).</p>
</blockquote>
<h4 id="main-loop">Main Loop</h4>
<p>The bridge runs at ~500 Hz and performs these operations each cycle:</p>
<ol>
<li><strong>Read IMU data</strong> from serial port</li>
<li><strong>Copy to shared memory</strong> (with semaphore lock)</li>
<li><strong>Publish LowState</strong> with motor + IMU data via DDS</li>
<li><strong>Process LowCmd</strong> commands from DDS</li>
</ol>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token comment">// Read IMU data into shared memory</span>
<span class="token keyword">if</span> <span class="token punctuation">(</span>imu_fd <span class="token operator">&gt;=</span> <span class="token number">0</span> <span class="token operator">&amp;&amp;</span> shm_data<span class="token punctuation">)</span> <span class="token punctuation">{</span>
  IMUReading reading<span class="token punctuation">;</span>
  <span class="token keyword">if</span> <span class="token punctuation">(</span><span class="token function">imu_serial_read</span><span class="token punctuation">(</span>imu_fd<span class="token punctuation">,</span> <span class="token operator">&amp;</span>reading<span class="token punctuation">)</span> <span class="token operator">&gt;</span> <span class="token number">0</span> <span class="token operator">&amp;&amp;</span> reading<span class="token punctuation">.</span>valid<span class="token punctuation">)</span> <span class="token punctuation">{</span>
    <span class="token function">lock_semaphore</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">memcpy</span><span class="token punctuation">(</span>shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>quaternion<span class="token punctuation">,</span> reading<span class="token punctuation">.</span>quaternion<span class="token punctuation">,</span> <span class="token keyword">sizeof</span><span class="token punctuation">(</span>reading<span class="token punctuation">.</span>quaternion<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">memcpy</span><span class="token punctuation">(</span>shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>gyroscope<span class="token punctuation">,</span> reading<span class="token punctuation">.</span>gyroscope<span class="token punctuation">,</span> <span class="token keyword">sizeof</span><span class="token punctuation">(</span>reading<span class="token punctuation">.</span>gyroscope<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">memcpy</span><span class="token punctuation">(</span>shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>accelerometer<span class="token punctuation">,</span> reading<span class="token punctuation">.</span>accelerometer<span class="token punctuation">,</span> <span class="token keyword">sizeof</span><span class="token punctuation">(</span>reading<span class="token punctuation">.</span>accelerometer<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    <span class="token function">memcpy</span><span class="token punctuation">(</span>shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>rpy<span class="token punctuation">,</span> reading<span class="token punctuation">.</span>rpy<span class="token punctuation">,</span> <span class="token keyword">sizeof</span><span class="token punctuation">(</span>reading<span class="token punctuation">.</span>rpy<span class="token punctuation">)</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
    shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>temperature <span class="token operator">=</span> reading<span class="token punctuation">.</span>temperature<span class="token punctuation">;</span>
    shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>timestamp_ns <span class="token operator">=</span> reading<span class="token punctuation">.</span>timestamp_us <span class="token operator">*</span> <span class="token number">1000</span><span class="token punctuation">;</span>
    shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>valid <span class="token operator">=</span> <span class="token boolean">true</span><span class="token punctuation">;</span>
    <span class="token function">unlock_semaphore</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token punctuation">}</span>
<span class="token punctuation">}</span>
</code></pre>
<h4 id="dds-publishing">DDS Publishing</h4>
<p>IMU data is included in the <code>LowState_</code> message:</p>
<pre class=" language-cpp"><code class="prism  language-cpp"><span class="token keyword">if</span> <span class="token punctuation">(</span>shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>valid<span class="token punctuation">)</span> <span class="token punctuation">{</span>
  <span class="token function">memcpy</span><span class="token punctuation">(</span>state<span class="token punctuation">.</span>imu_state<span class="token punctuation">.</span>quaternion<span class="token punctuation">,</span> shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>quaternion<span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">memcpy</span><span class="token punctuation">(</span>state<span class="token punctuation">.</span>imu_state<span class="token punctuation">.</span>gyroscope<span class="token punctuation">,</span> shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>gyroscope<span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">memcpy</span><span class="token punctuation">(</span>state<span class="token punctuation">.</span>imu_state<span class="token punctuation">.</span>accelerometer<span class="token punctuation">,</span> shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>accelerometer<span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  <span class="token function">memcpy</span><span class="token punctuation">(</span>state<span class="token punctuation">.</span>imu_state<span class="token punctuation">.</span>rpy<span class="token punctuation">,</span> shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>rpy<span class="token punctuation">,</span> <span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">.</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
  state<span class="token punctuation">.</span>imu_state<span class="token punctuation">.</span>temperature <span class="token operator">=</span> shm_data<span class="token operator">-</span><span class="token operator">&gt;</span>imu<span class="token punctuation">.</span>temperature<span class="token punctuation">;</span>
<span class="token punctuation">}</span>
</code></pre>
<hr>
<h2 id="usage">Usage</h2>
<h3 id="running-the-system">Running the System</h3>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Terminal 1: Start RT thread (requires sudo for real-time permissions)</span>
<span class="token function">cd</span> /home/yctan/g1_motor_control/build
<span class="token function">sudo</span> ./bin/ec_rt_thread

<span class="token comment"># Terminal 2: Start Unitree bridge (reads IMU and publishes DDS)</span>
./bin/unitree_bridge
</code></pre>
<h3 id="verifying-imu-data">Verifying IMU Data</h3>
<p>Check that the IMU is detected during startup:</p>
<pre><code>[IMU] Opened /dev/ttyUSB0 at 460800 baud
</code></pre>
<p>If not connected:</p>
<pre><code>Warning: IMU not available, continuing without IMU
</code></pre>
<hr>
<h2 id="file-summary">File Summary</h2>

<table>
<thead>
<tr>
<th>File</th>
<th>Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td>[imu_serial.h](file:///home/yctan/g1_motor_control/src/imu_serial.h)</td>
<td>IMU driver interface</td>
</tr>
<tr>
<td>[imu_serial.c](file:///home/yctan/g1_motor_control/src/imu_serial.c)</td>
<td>Yesense protocol parser (380 lines)</td>
</tr>
<tr>
<td>[ec_shared_mem.h](file:///home/yctan/g1_motor_control/src/ec_shared_mem.h)</td>
<td>IMUData structure definition</td>
</tr>
<tr>
<td>[unitree_bridge.cpp](file:///home/yctan/g1_motor_control/src/unitree_bridge.cpp)</td>
<td>Integration with DDS publishing</td>
</tr>
<tr>
<td>[CMakeLists.txt](file:///home/yctan/g1_motor_control/src/CMakeLists.txt)</td>
<td>Build configuration (lines 130-157)</td>
</tr>
</tbody>
</table>
