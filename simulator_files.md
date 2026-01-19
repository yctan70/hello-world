---


---

<h1 id="shared-memory-simulator---complete-file-summary">Shared Memory Simulator - Complete File Summary</h1>
<h2 id="overview">Overview</h2>
<p>This simulator enables testing <code>ls_motor.c</code> without EtherCAT hardware or libfakeethercat. It uses POSIX shared memory for direct PDO exchange.</p>
<hr>
<h2 id="core-simulator-files">Core Simulator Files</h2>
<h3 id="shm_sim.c-332-lines">1. shm_sim.c (332 lines)</h3>
<p><strong>Purpose</strong>: Standalone EtherCAT simulator with CiA 402 state machine</p>
<p><strong>Key Features</strong>:</p>
<ul>
<li>POSIX shared memory interface (<code>/ethercat_sim_pdo</code>)</li>
<li>Full motor state machine (6 states: NOT_READY → SWITCH_ON_DISABLED → READY_TO_SWITCH_ON → SWITCHED_ON → OPERATION_ENABLED → QUICK_STOP)</li>
<li>Simulates motor physics (position, velocity, torque)</li>
<li>Mutex-protected thread-safe access</li>
<li>10ms cycle time (configurable)</li>
</ul>
<p><strong>Compilation</strong>:</p>
<pre class=" language-bash"><code class="prism  language-bash">gcc -Wall -O2 -g -o shm_sim shm_sim.c -lpthread -lrt
</code></pre>
<p><strong>Usage</strong>:</p>
<pre class=" language-bash"><code class="prism  language-bash">./shm_sim
</code></pre>
<p><strong>Output</strong>:</p>
<pre><code>Cycle   300: State=2, CW=0x0006, SW=0x0221, Pos=0, Vel=0, Trq=0
Motor: State transition 2 -&gt; 3 (CW: 0x0007)
</code></pre>
<hr>
<h3 id="ecrt_shm_shim.c-230-lines">2. ecrt_shm_shim.c (230 lines)</h3>
<p><strong>Purpose</strong>: EtherCAT API shim library that redirects ecrt_* calls to shared memory</p>
<p><strong>Implements</strong>:</p>
<ul>
<li><code>ecrt_request_master()</code> - Returns dummy pointer</li>
<li><code>ecrt_master_create_domain()</code> - Returns dummy pointer</li>
<li><code>ecrt_slave_config_pdos()</code> - Logs configuration</li>
<li><code>ecrt_domain_reg_pdo_entry_list()</code> - Maps PDO offsets</li>
<li><code>ecrt_master_activate()</code> - <strong>Opens shared memory</strong> (<code>shm_open</code>, <code>mmap</code>)</li>
<li><code>ecrt_domain_process()</code> - <strong>Reads from SHM</strong> (Status Word, Position, Velocity, Torque)</li>
<li><code>ecrt_domain_queue()</code> - <strong>Writes to SHM</strong> (Control Word, Target Torque, Mode)</li>
<li><code>ecrt_master_send/receive()</code> - No-op</li>
<li>SDO stubs (return success)</li>
</ul>
<p><strong>Compilation</strong>:</p>
<pre class=" language-bash"><code class="prism  language-bash">gcc -shared -fPIC -Wall -O2 -I/usr/local/include \
    -o libecrt_shm.so ecrt_shm_shim.c -lpthread -lrt
</code></pre>
<p><strong>Result</strong>: <code>libecrt_shm.so</code> (shared library)</p>
<hr>
<h3 id="shared-memory-structure">3. Shared Memory Structure</h3>
<p>Defined in both <code>shm_sim.c</code> and <code>ecrt_shm_shim.c</code>:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token keyword">typedef</span> <span class="token keyword">struct</span> <span class="token punctuation">{</span>
    <span class="token comment">// Master → Simulator (3 bytes + padding)</span>
    uint16_t control_word<span class="token punctuation">;</span>      <span class="token comment">// Offset 0</span>
    int16_t target_torque<span class="token punctuation">;</span>      <span class="token comment">// Offset 2</span>
    int8_t operation_mode<span class="token punctuation">;</span>      <span class="token comment">// Offset 4</span>
    
    // Simulator → Master (18 bytes)
    uint16_t error_code;  // Read-only
    uint16_t status_word; // Offset 5
    int8_t mode_display; // Offset 7
    int32_t actual_position; // Offset 8
    int32_t actual_velocity; // Offset 12
    int32_t following_error; // Read-only
    int16_t actual_torque; // Offset 16
    
    <span class="token comment">// Synchronization</span>
    pthread_mutex_t mutex<span class="token punctuation">;</span>
    uint32_t master_cycle_count<span class="token punctuation">;</span>
    uint32_t sim_cycle_count<span class="token punctuation">;</span>
    bool initialized<span class="token punctuation">;</span>
<span class="token punctuation">}</span> shm_pdo_t<span class="token punctuation">;</span>  <span class="token comment">// Total: 83 bytes</span>
</code></pre>
<p><strong>Location</strong>: <code>/dev/shm/ethercat_sim_pdo</code></p>
<hr>
<h2 id="test-utilities">Test Utilities</h2>
<h3 id="shm_wrapper.h-107-lines">4. shm_wrapper.h (107 lines)</h3>
<p><strong>Purpose</strong>: Simplified API for C applications (alternative to shim library)</p>
<p><strong>Functions</strong>:</p>
<ul>
<li><code>int shm_init()</code> - Open shared memory</li>
<li><code>void shm_read(...)</code> - Read all inputs</li>
<li><code>void shm_write(...)</code> - Write all outputs</li>
<li><code>void shm_cleanup()</code> - Unmap and close</li>
<li><code>void shm_print_stats()</code> - Print cycle counts</li>
</ul>
<p><strong>Usage</strong>:</p>
<pre class=" language-c"><code class="prism  language-c"><span class="token macro property">#<span class="token directive keyword">include</span> <span class="token string">"shm_wrapper.h"</span></span>

<span class="token function">shm_init</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">shm_write</span><span class="token punctuation">(</span><span class="token number">0x0006</span><span class="token punctuation">,</span> <span class="token number">0</span><span class="token punctuation">,</span> <span class="token number">10</span><span class="token punctuation">)</span><span class="token punctuation">;</span>  <span class="token comment">// Shutdown command</span>
<span class="token function">shm_read</span><span class="token punctuation">(</span><span class="token operator">&amp;</span>status<span class="token punctuation">,</span> <span class="token operator">&amp;</span>mode<span class="token punctuation">,</span> <span class="token operator">&amp;</span>pos<span class="token punctuation">,</span> <span class="token operator">&amp;</span>vel<span class="token punctuation">,</span> <span class="token operator">&amp;</span>trq<span class="token punctuation">)</span><span class="token punctuation">;</span>
<span class="token function">shm_cleanup</span><span class="token punctuation">(</span><span class="token punctuation">)</span><span class="token punctuation">;</span>
</code></pre>
<hr>
<h3 id="shm_test_client.c-generated-by-test_shm.sh">5. shm_test_client.c (Generated by test_shm.sh)</h3>
<p><strong>Purpose</strong>: Standalone test client demonstrating servo enable sequence</p>
<p><strong>Test Sequence</strong>:</p>
<ol>
<li>Shutdown (0x0006) → Wait for Ready to Switch On (0x0221)</li>
<li>Switch On (0x0007) → Wait for Switched On (0x0223)</li>
<li>Enable Operation (0x000F) → Wait for Operation Enabled (0x0627)</li>
<li>Apply torque commands</li>
</ol>
<p><strong>Compilation</strong>:</p>
<pre class=" language-bash"><code class="prism  language-bash">gcc -Wall -O2 -o shm_test_client shm_test_client.c -lpthread -lrt
</code></pre>
<hr>
<h2 id="integration-scripts">Integration Scripts</h2>
<h3 id="test_ls_motor_shm.sh-55-lines">6. test_ls_motor_shm.sh (55 lines)</h3>
<p><strong>Purpose</strong>: Automated test script for <code>ls_motor.c</code> with shim</p>
<p><strong>Steps</strong>:</p>
<ol>
<li>Build <code>libecrt_shm.so</code> from <code>ecrt_shm_shim.c</code></li>
<li>Compile <code>ls_motor.c</code> → <code>ls_motor_shm</code> (linked with shim)</li>
<li>Start <code>shm_sim</code> in background</li>
<li>Run <code>ls_motor_shm 1</code></li>
<li>Check for success keywords</li>
<li>Display logs</li>
</ol>
<p><strong>Usage</strong>:</p>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token function">chmod</span> +x test_ls_motor_shm.sh
./test_ls_motor_shm.sh
</code></pre>
<p><strong>Output</strong>:</p>
<pre><code>✅ SUCCESS - Motor enabled!
</code></pre>
<hr>
<h3 id="test_shm.sh-older-version-80-lines">7. test_shm.sh (Older version, 80 lines)</h3>
<p><strong>Purpose</strong>: Test script for standalone client (not ls_motor)</p>
<p>Creates and runs <code>shm_test_client.c</code> inline.</p>
<hr>
<h2 id="documentation">Documentation</h2>
<h3 id="testing.md">8. <a href="http://TESTING.md">TESTING.md</a></h3>
<p><strong>Purpose</strong>: User guide for manual and automated testing</p>
<p><strong>Sections</strong>:</p>
<ul>
<li>Quick Start</li>
<li>Manual Testing (2 terminals)</li>
<li>How It Works (architecture diagram)</li>
<li>Files Created</li>
<li>Cleanup</li>
<li>Troubleshooting</li>
</ul>
<hr>
<h2 id="file-summary-table">File Summary Table</h2>

<table>
<thead>
<tr>
<th>File</th>
<th>Lines</th>
<th>Purpose</th>
<th>Compiles To</th>
</tr>
</thead>
<tbody>
<tr>
<td><code>shm_sim.c</code></td>
<td>332</td>
<td>Standalone simulator</td>
<td><code>shm_sim</code> (28KB)</td>
</tr>
<tr>
<td><code>ecrt_shm_shim.c</code></td>
<td>230</td>
<td>EtherCAT API shim</td>
<td><code>libecrt_shm.so</code> (18KB)</td>
</tr>
<tr>
<td><code>shm_wrapper.h</code></td>
<td>107</td>
<td>Simplified API</td>
<td>Header only</td>
</tr>
<tr>
<td><code>shm_test_client.c</code></td>
<td>~120</td>
<td>Test client</td>
<td><code>shm_test_client</code> (17KB)</td>
</tr>
<tr>
<td><code>test_ls_motor_shm.sh</code></td>
<td>55</td>
<td>Automated test</td>
<td>Script</td>
</tr>
<tr>
<td><code>test_shm.sh</code></td>
<td>80</td>
<td>Client test</td>
<td>Script</td>
</tr>
<tr>
<td><code>TESTING.md</code></td>
<td>-</td>
<td>Documentation</td>
<td>-</td>
</tr>
</tbody>
</table><hr>
<h2 id="complete-build-and-test-commands">Complete Build and Test Commands</h2>
<pre class=" language-bash"><code class="prism  language-bash"><span class="token comment"># 1. Build simulator</span>
gcc -Wall -O2 -g -o shm_sim shm_sim.c -lpthread -lrt

<span class="token comment"># 2. Build shim library</span>
gcc -shared -fPIC -Wall -O2 -I/usr/local/include \
    -o libecrt_shm.so ecrt_shm_shim.c -lpthread -lrt

<span class="token comment"># 3. Compile ls_motor with shim</span>
gcc -Wall -O2 -g -D_GNU_SOURCE -I/usr/local/include \
    -o ls_motor_shm ls_motor.c \
    -L. -lecrt_shm -lpthread -lrt -lm -Wl,-rpath,.

<span class="token comment"># 4. Test</span>
./shm_sim <span class="token operator">&amp;</span>           <span class="token comment"># Terminal 1</span>
./ls_motor_shm 1      <span class="token comment"># Terminal 2</span>
</code></pre>
<p><strong>Or use automated script</strong>:</p>
<pre class=" language-bash"><code class="prism  language-bash">./test_ls_motor_shm.sh
</code></pre>
<hr>
<h2 id="data-flow-architecture">Data Flow Architecture</h2>
<pre><code>┌─────────────────────────────────────────────────────┐
│ ls_motor.c (unchanged)                              │
│   - Calls ecrt_domain_process()                     │
│   - Calls ecrt_domain_queue()                       │
│   - Reads/writes domain_pd buffer                   │
└────────────────┬────────────────────────────────────┘
                 │ ecrt_* API calls
                 ▼
┌─────────────────────────────────────────────────────┐
│ libecrt_shm.so (shim)                               │
│   - Intercepts ecrt_* functions                     │
│   - Maintains local domain_pd buffer                │
│   - process(): SHM → domain_pd                      │
│   - queue(): domain_pd → SHM                        │
└────────────────┬────────────────────────────────────┘
                 │ pthread_mutex_lock/unlock
                 │ read/write shm_pdo_t
                 ▼
┌─────────────────────────────────────────────────────┐
│ /dev/shm/ethercat_sim_pdo (83 bytes)                │
│   - control_word, target_torque, operation_mode     │
│   - status_word, position, velocity, torque         │
│   - pthread_mutex (process-shared)                  │
└────────────────┬────────────────────────────────────┘
                 │ pthread_mutex_lock/unlock
                 │ read/write shm_pdo_t
                 ▼
┌─────────────────────────────────────────────────────┐
│ shm_sim (standalone)                                │
│   - Reads control_word                              │
│   - Updates state machine                           │
│   - Simulates motor physics                         │
│   - Writes status_word, position, velocity, torque  │
└─────────────────────────────────────────────────────┘
</code></pre>
<hr>
<h2 id="key-advantages">Key Advantages</h2>
<p>✅ <strong>No libfakeethercat</strong> - Bypasses RtIPC complexity<br>
✅ <strong>No code changes</strong> - ls_motor.c works unmodified<br>
✅ <strong>Simple debugging</strong> - Direct memory access, clear flow<br>
✅ <strong>Fast</strong> - Sub-millisecond latency<br>
✅ <strong>Portable</strong> - Standard POSIX, works anywhere<br>
✅ <strong>Proven</strong> - Successfully enables servo (0x0627 status)</p>
<hr>
<h2 id="logs-and-output-files">Logs and Output Files</h2>
<p>Generated during testing:</p>
<ul>
<li><code>ls_motor_test.log</code> - Master application output</li>
<li><code>ls_motor_sim.log</code> - Simulator output</li>
<li><code>shm_sim.log</code> - Simulator debug log</li>
<li><code>client_verify.log</code> - Test client results</li>
<li><code>sim_verify.log</code> - Verification run logs</li>
</ul>

