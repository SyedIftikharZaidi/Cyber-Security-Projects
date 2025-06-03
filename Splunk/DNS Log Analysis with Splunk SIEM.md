<h1>DNS Log Analysis with Splunk SIEM</h1>

<h2>Project Overview</h2>
<p>This project demonstrates how to analyze DNS logs using Splunk SIEM to monitor network activity and detect potential security threats.</p>

<h2>Prerequisites</h2>
<ul>
  <li><strong>Splunk Enterprise</strong>: Installed and configured with proper licensing</li>
  <li><strong>DNS Log Forwarding</strong>: Configure your DNS servers to send logs to Splunk</li>
  <li><strong>SPL Basics</strong>: Familiarity with Splunk's search syntax</li>
</ul>


<h2>Setup Intstructions</h2>
<h4>Step 1: Prepare DNS Log Files</h4>
<p>Ensure your log files contain these fields in text format:</p>
<ul>
  <li>Source and destination IP addresses</li>
  <li>Domain names (FQDNs)</li>
  <li>DNS query types (A, AAAA, MX, TXT, etc.)</li>
  <li>Response codes (NOERROR, NXDOMAIN, SERVFAIL, etc.)</li>
</ul>

<h4>Step 2: Access Splunk Web Interface</h4>
<ol>
  <li>Log in to your Splunk instance</li>
  <li>Navigate to: <strong>Settings</strong> → <strong>Add Data</strong> → <strong>Upload</strong></li>
</ol>

<h4>Step 3: Select Log File</h4>
<ul>
  <li>Click <strong>Select File</strong> and choose your DNS log file</li>
  <li>Supported formats: <code>.log</code>, <code>.txt</code>, or <code>.gz</code> compressed files</li>
</ul>

<h4>Step 4: Configure Input Settings</h4>
<ol>
  <li><strong>Source Type</strong>: Select <code>dns</code> (or create a custom one if needed)</li>
  <li><strong>Index</strong>:
    <ul>
      <li>Specify a dedicated index (e.g., <code>dns_logs</code>)</li>
      <li>Or use the default <code>main</code> index</li>
    </ul>
  </li>
  <li><strong>Host</strong>:
    <ul>
      <li>Auto-populated if logs contain hostnames</li>
      <li>Manually set if needed (e.g., <code>dns-server-01</code>)</li>
    </ul>
  </li>
</ol>

<h4>Step 5: Verify Successful Upload</h4>
<p>Run this search to confirm data is ingested:</p>
<pre><code class="language-splunk">index=&lt;your_dns_index&gt; sourcetype=&lt;your_dns_sourcetype&gt;</code></pre>
<p>Replace placeholders with your actual values.</p>

<h4>Troubleshooting Tips</h4>
<ul>
  <li>If no results appear:
    <ul>
      <li>Check <strong>Settings</strong> → <strong>Data inputs</strong> to verify the upload</li>
      <li>Confirm the index has read permissions</li>
      <li>Validate timestamp extraction in <strong>Settings</strong> → <strong>Source types</strong></li>
    </ul>
  </li>
</ul>

<h2>DNS Analysis Queries</h2>

<h3>Basic Queries</h3>

<h4>1. Count DNS queries by type</h4>
<pre><code class="language-splunk"><span style="color:#7dcfff">index=</span><span style="color:#ff9d00">&lt;dns_index&gt;</span> <span style="color:#7dcfff">sourcetype=</span><span style="color:#ff9d00">&lt;dns_sourcetype&gt;</span>
<span style="color:#7dcfff">| stats count by query_type</span>
<span style="color:#7dcfff">| sort - count</span>
</code></pre>
<p><strong>What this does:</strong><br>
- Lists all DNS query types (A, AAAA, TXT, etc.)<br>
- Counts how many times each type appears<br>
- Sorts results from highest to lowest<br>
<strong>Use case:</strong> Identify unusual query types (e.g., unexpected TXT queries)</p>

<h4>2. Top queried domains</h4>
<pre><code class="language-splunk"><span style="color:#7dcfff">index=</span><span style="color:#ff9d00">&lt;dns_index&gt;</span> 
<span style="color:#7dcfff">| top limit=20 fqdn</span>
</code></pre>
<p><strong>What this does:</strong><br>
- Shows the 20 most frequently queried domains<br>
- Automatically calculates percentages<br>
<strong>Use case:</strong> Spot trending domains or potential beaconing</p>

<h3>Threat Detection Queries</h3>

<h4>3. Detect suspicious queries</h4>
<pre><code class="language-splunk"><span style="color:#7dcfff">index=</span><span style="color:#ff9d00">&lt;dns_index&gt;</span> 
<span style="color:#7dcfff">| regex fqdn="(exe|dll|bat|ps1)$"</span>
<span style="color:#7dcfff">| table _time, src_ip, fqdn</span>
</code></pre>
<p><strong>What this does:</strong><br>
- Finds domains ending with executable extensions<br>
- Returns timestamp, source IP, and full domain<br>
<strong>Use case:</strong> Detect potential malware C2 callbacks</p>

<h4>4. DNS tunneling indicators</h4>
<pre><code class="language-splunk"><span style="color:#7dcfff">index=</span><span style="color:#ff9d00">&lt;dns_index&gt;</span>
<span style="color:#7dcfff">| where len(fqdn) > 100</span>
<span style="color:#7dcfff">| stats count by src_ip, fqdn</span>
</code></pre>
<p><strong>What this does:</strong><br>
- Flags unusually long domain names (>100 chars)<br>
- Groups results by source IP and domain<br>
<strong>Use case:</strong> Identify possible data exfiltration via DNS</p>
