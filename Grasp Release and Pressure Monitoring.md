---


---

<h2 id="grasprelease-and-pressure-monitoring-walkthrough">Grasp/Release and Pressure Monitoring Walkthrough</h2>
<p>This document describes the changes made to <code>ls_hand_pro.c</code>  and <code>ls_hand.c</code>.</p>
<h3 id="ls_hand_pro.c--using-lhandprolib"><code>ls_hand_pro.c</code>  (Using LHandProLib)</h3>
<ul>
<li>Implemented a periodic grasp/release cycle every 3 seconds.</li>
<li>Added pressure monitoring for 5 fingertips.</li>
<li>Build:  <code>make ls_hand_pro</code></li>
<li>Run:  <code>sudo ./ls_hand_pro</code></li>
</ul>
<h3 id="ls_hand.c--no-library"><code>ls_hand.c</code>  (No Library)</h3>
<ul>
<li>Ported the same logic to use direct <em>EtherCAT</em> commands.</li>
<li>Handles raw <em>PDO</em> data for axis control and sensor feedback.</li>
<li><strong>Note</strong>: Uses 0-based indexing for axes (0-5), unlike the library’s 1-based indexing.</li>
<li>Build:  <code>make ls_hand</code></li>
<li>Run:  <code>sudo ./ls_hand</code></li>
</ul>
<h3 id="expected-behavior-both-versions">Expected Behavior (Both Versions)</h3>
<p>The hand will cycle between:</p>
<ol>
<li><strong>Grasp</strong>: All fingers move to position 10000.</li>
<li><strong>Release</strong>: All fingers move to position 0.</li>
</ol>
<p>The terminal will confirm operation:</p>
<pre><code>Cycle 1234 | OP: YES | ... | State: CLOSING/HOLDING
...
Thumb :   0.50 N | Index :   1.20 N | ...
</code></pre>

