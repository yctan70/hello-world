---


---

<h1 id="motor-test-gui-walkthrough">Motor Test GUI Walkthrough</h1>
<h2 id="summary">Summary</h2>
<p>Built a unified motor control system for SE and LS motors with:</p>
<ul>
<li>Mock EtherCAT API for simulation</li>
<li>Direct shared memory GUI for fast response</li>
<li>DDS bridge for future ROS2 integration</li>
</ul>
<h2 id="architecture">Architecture</h2>
<pre class=" language-mermaid"><svg id="mermaid-svg-1NLDwmMhULuOR9rp" width="100%" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" height="588.03125" style="max-width: 532.55859375px;" viewBox="0 0 532.55859375 588.03125"><style>#mermaid-svg-1NLDwmMhULuOR9rp{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;fill:#000000;}#mermaid-svg-1NLDwmMhULuOR9rp .error-icon{fill:#552222;}#mermaid-svg-1NLDwmMhULuOR9rp .error-text{fill:#552222;stroke:#552222;}#mermaid-svg-1NLDwmMhULuOR9rp .edge-thickness-normal{stroke-width:2px;}#mermaid-svg-1NLDwmMhULuOR9rp .edge-thickness-thick{stroke-width:3.5px;}#mermaid-svg-1NLDwmMhULuOR9rp .edge-pattern-solid{stroke-dasharray:0;}#mermaid-svg-1NLDwmMhULuOR9rp .edge-pattern-dashed{stroke-dasharray:3;}#mermaid-svg-1NLDwmMhULuOR9rp .edge-pattern-dotted{stroke-dasharray:2;}#mermaid-svg-1NLDwmMhULuOR9rp .marker{fill:#666;stroke:#666;}#mermaid-svg-1NLDwmMhULuOR9rp .marker.cross{stroke:#666;}#mermaid-svg-1NLDwmMhULuOR9rp svg{font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:16px;}#mermaid-svg-1NLDwmMhULuOR9rp .label{font-family:"trebuchet ms",verdana,arial,sans-serif;color:#000000;}#mermaid-svg-1NLDwmMhULuOR9rp .cluster-label text{fill:#333;}#mermaid-svg-1NLDwmMhULuOR9rp .cluster-label span{color:#333;}#mermaid-svg-1NLDwmMhULuOR9rp .label text,#mermaid-svg-1NLDwmMhULuOR9rp span{fill:#000000;color:#000000;}#mermaid-svg-1NLDwmMhULuOR9rp .node rect,#mermaid-svg-1NLDwmMhULuOR9rp .node circle,#mermaid-svg-1NLDwmMhULuOR9rp .node ellipse,#mermaid-svg-1NLDwmMhULuOR9rp .node polygon,#mermaid-svg-1NLDwmMhULuOR9rp .node path{fill:#eee;stroke:#999;stroke-width:1px;}#mermaid-svg-1NLDwmMhULuOR9rp .node .label{text-align:center;}#mermaid-svg-1NLDwmMhULuOR9rp .node.clickable{cursor:pointer;}#mermaid-svg-1NLDwmMhULuOR9rp .arrowheadPath{fill:#333333;}#mermaid-svg-1NLDwmMhULuOR9rp .edgePath .path{stroke:#666;stroke-width:1.5px;}#mermaid-svg-1NLDwmMhULuOR9rp .flowchart-link{stroke:#666;fill:none;}#mermaid-svg-1NLDwmMhULuOR9rp .edgeLabel{background-color:white;text-align:center;}#mermaid-svg-1NLDwmMhULuOR9rp .edgeLabel rect{opacity:0.5;background-color:white;fill:white;}#mermaid-svg-1NLDwmMhULuOR9rp .cluster rect{fill:hsl(210,66.6666666667%,95%);stroke:#26a;stroke-width:1px;}#mermaid-svg-1NLDwmMhULuOR9rp .cluster text{fill:#333;}#mermaid-svg-1NLDwmMhULuOR9rp .cluster span{color:#333;}#mermaid-svg-1NLDwmMhULuOR9rp div.mermaidTooltip{position:absolute;text-align:center;max-width:200px;padding:2px;font-family:"trebuchet ms",verdana,arial,sans-serif;font-size:12px;background:hsl(-160,0%,93.3333333333%);border:1px solid #26a;border-radius:2px;pointer-events:none;z-index:100;}#mermaid-svg-1NLDwmMhULuOR9rp:root{--mermaid-font-family:"trebuchet ms",verdana,arial,sans-serif;}#mermaid-svg-1NLDwmMhULuOR9rp flowchart-v2{fill:apa;}</style><g transform="translate(0, 0)"><marker id="flowchart-pointEnd" class="marker flowchart" viewBox="0 0 10 10" refX="9" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="flowchart-pointStart" class="marker flowchart" viewBox="0 0 10 10" refX="0" refY="5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 0 5 L 10 10 L 10 0 z" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="flowchart-circleEnd" class="marker flowchart" viewBox="0 0 10 10" refX="11" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="flowchart-circleStart" class="marker flowchart" viewBox="0 0 10 10" refX="-1" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" class="arrowMarkerPath" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="flowchart-crossEnd" class="marker cross flowchart" viewBox="0 0 11 11" refX="12" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="flowchart-crossStart" class="marker cross flowchart" viewBox="0 0 11 11" refX="-1" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" class="arrowMarkerPath" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><g class="root"><g class="clusters"><g class="cluster default" id="MOCK"><rect style="" rx="0" ry="0" x="253.73046875" y="461.59375" width="270.828125" height="118.4375"></rect><g class="cluster-label" transform="translate(351.79296875, 466.59375)"><foreignObject width="74.703125" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Simulation</span></div></foreignObject></g></g><g class="cluster default" id="RT"><rect style="" rx="0" ry="0" x="24.2421875" y="318.15625" width="209.48828125" height="261.875"></rect><g class="cluster-label" transform="translate(70.150390625, 323.15625)"><foreignObject width="117.671875" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Real-Time Layer</span></div></foreignObject></g></g><g class="cluster default" id="SHM"><rect style="" rx="0" ry="0" x="8" y="176.4375" width="260.1875" height="91.71875"></rect><g class="cluster-label" transform="translate(81.1796875, 181.4375)"><foreignObject width="113.828125" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Shared Memory</span></div></foreignObject></g></g><g class="cluster default" id="GUI"><rect style="" rx="0" ry="0" x="49.34375" y="8" width="177.5" height="91.71875"></rect><g class="cluster-label" transform="translate(96.7421875, 13)"><foreignObject width="82.703125" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">Python GUI</span></div></foreignObject></g></g></g><g class="edgePaths"><path d="M138.09375,74.71875L138.09375,78.88541666666667C138.09375,83.05208333333333,138.09375,91.38541666666667,138.09375,101.9453125C138.09375,112.50520833333333,138.09375,125.29166666666667,138.09375,138.078125C138.09375,150.86458333333334,138.09375,163.65104166666666,138.09375,174.2109375C138.09375,184.77083333333334,138.09375,193.10416666666666,138.09375,197.27083333333334L138.09375,201.4375" id="L-motor_gui-shm" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-motor_gui LE-shm" style="fill:none;" marker-end="url(#flowchart-pointEnd)"></path><path d="M138.09375,243.15625L138.09375,247.32291666666666C138.09375,251.48958333333334,138.09375,259.8229166666667,138.09375,268.15625C138.09375,276.4895833333333,138.09375,284.8229166666667,138.09375,293.15625C138.09375,301.4895833333333,138.09375,309.8229166666667,138.09375,318.15625C138.09375,326.4895833333333,138.09375,334.8229166666667,138.09375,338.9895833333333L138.09375,343.15625" id="L-shm-ec_rt" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-shm LE-ec_rt" style="fill:none;" marker-end="url(#flowchart-pointEnd)"></path><path d="M132.315385883905,411.59375L131.6117799032542,415.7604166666667C130.90817392260334,419.9270833333333,129.50096196130167,428.2604166666667,128.79735598065085,436.59375C128.09375,444.9270833333333,128.09375,453.2604166666667,128.09375,461.59375C128.09375,469.9270833333333,128.09375,478.2604166666667,128.09375,482.4270833333333L128.09375,486.59375" id="L-ec_rt-dds" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-ec_rt LE-dds" style="fill:none;" marker-start="url(#flowchart-pointStart)" marker-end="url(#flowchart-pointEnd)"></path><path d="M162.3199929914248,411.59375L165.26991603452066,415.7604166666667C168.21983907761654,419.9270833333333,174.11968516380827,428.2604166666667,177.06960820690415,436.59375C180.01953125,444.9270833333333,180.01953125,453.2604166666667,198.13802083333334,462.5577673979503C216.25651041666666,471.8551181292339,252.49348958333334,482.11648625846783,270.6119791666667,487.24717032308473L288.73046875,492.37785438770175" id="L-ec_rt-ecrt_mock" class=" edge-thickness-normal edge-pattern-solid flowchart-link LS-ec_rt LE-ecrt_mock" style="fill:none;" marker-end="url(#flowchart-pointEnd)"></path></g><g class="edgeLabels"><g class="edgeLabel" transform="translate(138.09375, 138.078125)"><g class="label" transform="translate(-56.0234375, -13.359375)"><foreignObject width="112.046875" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel">posix_ipc (fast!)</span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g><g class="edgeLabel"><g class="label" transform="translate(0, 0)"><foreignObject width="0" height="0"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="edgeLabel"></span></div></foreignObject></g></g></g><g class="nodes"><g class="node default default" id="flowchart-ecrt_mock-181" transform="translate(389.14453125, 520.8125)"><rect class="basic label-container" style="" rx="0" ry="0" x="-100.4140625" y="-34.21875" width="200.828125" height="68.4375"></rect><g class="label" style="" transform="translate(-92.9140625, -26.71875)"><foreignObject width="185.828125" height="53.4375"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">ecrt_mock<br>(motor physics simulation)</span></div></foreignObject></g></g><g class="node default default" id="flowchart-ec_rt-179" transform="translate(138.09375, 377.375)"><rect class="basic label-container" style="" rx="0" ry="0" x="-52.421875" y="-34.21875" width="104.84375" height="68.4375"></rect><g class="label" style="" transform="translate(-44.921875, -26.71875)"><foreignObject width="89.84375" height="53.4375"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">ec_rt_thread<br>(real-time)</span></div></foreignObject></g></g><g class="node default default" id="flowchart-dds-180" transform="translate(128.09375, 520.8125)"><rect class="basic label-container" style="" rx="0" ry="0" x="-68.8515625" y="-34.21875" width="137.703125" height="68.4375"></rect><g class="label" style="" transform="translate(-61.3515625, -26.71875)"><foreignObject width="122.703125" height="53.4375"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">dds_bridge<br>(for ROS2 future)</span></div></foreignObject></g></g><g class="node default default" id="flowchart-shm-178" transform="translate(138.09375, 222.296875)"><rect class="basic label-container" style="" rx="0" ry="0" x="-95.09375" y="-20.859375" width="190.1875" height="41.71875"></rect><g class="label" style="" transform="translate(-87.59375, -13.359375)"><foreignObject width="175.1875" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">/dev/shm/ec_motor_shm</span></div></foreignObject></g></g><g class="node default default" id="flowchart-motor_gui-177" transform="translate(138.09375, 53.859375)"><rect class="basic label-container" style="" rx="0" ry="0" x="-53.75" y="-20.859375" width="107.5" height="41.71875"></rect><g class="label" style="" transform="translate(-46.25, -13.359375)"><foreignObject width="92.5" height="26.71875"><div xmlns="http://www.w3.org/1999/xhtml" style="display: inline-block; white-space: nowrap;"><span class="nodeLabel">motor_gui.py</span></div></foreignObject></g></g></g></g></g></svg></pre>
<h2 id="key-files">Key Files</h2>

<table>
<thead>
<tr>
<th>Component</th>
<th>File</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
<td>Mock EtherCAT</td>
<td><code>src/ecrt_mock.c</code></td>
<td>Simulates motor physics</td>
</tr>
<tr>
<td>RT Thread</td>
<td><code>src/ec_rt_thread.c</code></td>
<td>Unified SE+LS control</td>
</tr>
<tr>
<td>Shared Memory</td>
<td><code>src/ec_shared_mem.h</code></td>
<td>IPC data structures</td>
</tr>
<tr>
<td>DDS Bridge</td>
<td><code>src/dds_bridge.cpp</code></td>
<td>For ROS2 integration</td>
</tr>
<tr>
<td>GUI</td>
<td><code>python/motor_gui.py</code></td>
<td>Direct SHM access</td>
</tr>
</tbody>
</table><h2 id="usage">Usage</h2>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># Build with mock EtherCAT</span>
<span class="token function">cd</span> build
cmake -DUSE_MOCK_ECAT<span class="token operator">=</span>ON <span class="token punctuation">..</span>
<span class="token function">make</span> -j<span class="token variable"><span class="token variable">$(</span>nproc<span class="token variable">)</span></span>

<span class="token comment"># Run RT thread (requires root for real-time scheduling)</span>
<span class="token function">sudo</span> ./bin/ec_rt_thread -s 2 -l 2

<span class="token comment"># Run GUI (direct shared memory - fast!)</span>
python3 python/motor_gui.py

<span class="token comment"># Optional: DDS bridge for ROS2 integration</span>
./bin/dds_bridge
</code></pre>
<h2 id="motor-types">Motor Types</h2>
<ul>
<li><strong>SE Motor (type=0)</strong>: Simplified protocol with direct impedance control</li>
<li><strong>LS Motor (type=1)</strong>: CiA 402 with CST mode, impedance control wrapper</li>
</ul>
<h2 id="control-parameters">Control Parameters</h2>
<ul>
<li><code>target_pos</code>: Target position (rad)</li>
<li><code>target_vel</code>: Target velocity (rad/s)</li>
<li><code>target_tff</code>: Feedforward torque (N·m)</li>
<li><code>KP</code>: Position gain</li>
<li><code>KD</code>: Velocity/damping gain</li>
</ul>
<h2 id="next-steps">Next Steps</h2>
<ul>
<li>ROS2 integration via DDS bridge</li>
<li>Humanoid robot joint control</li>
<li>Real EtherCAT hardware testing (set USE_MOCK_ECAT=OFF)</li>
</ul>

