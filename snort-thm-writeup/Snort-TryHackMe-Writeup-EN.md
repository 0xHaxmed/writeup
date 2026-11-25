# TryHackMe — Snort (Complete Walkthrough & Reference Guide)

> **Author:** Mohamed Hesham Mohamed (`0xhaxmed`)  
> **Platform:** TryHackMe  
> **Room:** Snort  
> **Focus:** IDS/IPS Fundamentals, Snort Operating Modes, PCAP Investigation, and Snort Rule Authoring  

---

## Table of Contents

- [Task 1: Introduction](#task-1-introduction)
- [Task 2: Interactive Material and VM](#task-2-interactive-material-and-vm)
- [Task 3: Introduction to IDS/IPS](#task-3-introduction-to-idsips)
- [Task 4: First Interaction with Snort](#task-4-first-interaction-with-snort)
- [Task 5: Operation Mode 1 — Sniffer Mode](#task-5-operation-mode-1--sniffer-mode)
- [Task 6: Operation Mode 2 — Packet Logger Mode](#task-6-operation-mode-2--packet-logger-mode)
- [Task 7: Operation Mode 3 — IDS/IPS Mode](#task-7-operation-mode-3--idsips-mode)
- [Task 8: Operation Mode 4 — PCAP Investigation](#task-8-operation-mode-4--pcap-investigation)
- [Task 9: Snort Rule Structure](#task-9-snort-rule-structure)
- [Task 10: Snort2 Operation Logic — Points to Remember](#task-10-snort2-operation-logic--points-to-remember)
- [Task 11: Conclusion & Best Practices](#task-11-conclusion--best-practices)
- [Quick Snort Cheatsheet](#quick-snort-cheatsheet)

---

## Task 1: Introduction

### What Is Snort and Why Do We Use It?

**Snort** is an open-source, rule-based **Network Intrusion Detection and Prevention System (NIDS/NIPS)** originally developed by Martin Roesch and maintained by open-source contributors alongside the Cisco Talos team.

Its core purpose is analyzing network traffic in real time (or from captured `.pcap` files), matching packets against human-written detection rules, and either:
- **Alerting and logging** suspicious activity (when acting as an **IDS**), or
- **Dropping/blocking** malicious packets inline (when acting as an **IPS**).

### Room Learning Objectives
1. Understanding the differences between IDS and IPS architectures.
2. Learning Snort's operating modes and CLI flags.
3. Writing custom Snort detection rules from scratch.
4. Investigating live traffic and PCAP captures using Snort.

### Questions & Answers

| Question | Answer |
| :--- | :--- |
| Read the task above. | `No answer needed` |

---

## Task 2: Interactive Material and VM

### Lab Environment Breakdown (What Is Where & Why It Helps)

Inside the lab machine's `Desktop/Task-Exercises` directory, the room separates practice material from the live system configuration:

1. **`Config-Sample/`**
   - **What it is:** Sample Snort configuration and rule files.
   - **Why it matters:** The installed Snort binary uses `/etc/snort` by default, so `Config-Sample` lets you inspect and experiment safely without breaking the system's working configuration.
2. **`Exercise-Files/`**
   - **What it is:** Task-specific folders (`TASK-6`, `TASK-8`, `TASK-9`, etc.) containing `.pcap`, `.log`, and `.rules` files.
   - **Why it matters:** Keeps each exercise's artifacts isolated so logs and rule outputs from different tasks never mix.
3. **`traffic-generator.sh`**
   - **What it is:** A script executed with `sudo ./traffic-generator.sh` that replays controlled exercise traffic on the local network interface while the VM is offline.
   - **Why it matters:** Snort needs active packets on the wire to demonstrate live sniffing, logging, and alerting. This script simulates predictable network activity for each task so you can verify your commands reliably.

> **Operational Tip:** Always start your Snort listener **first**, then run `sudo ./traffic-generator.sh`, select the matching exercise, and wait until traffic generation completes before stopping Snort with `CTRL+C`.

### Questions & Answers

| Question | How We Got It | Answer |
| :--- | :--- | :--- |
| Navigate to the `Task-Exercises` folder and run the command `./.easy.sh` and write the output | Executed `./.easy.sh` inside `Task-Exercises` to verify basic terminal navigation in the lab environment. | `Too Easy!` |

---

## Task 3: Introduction to IDS/IPS

Before running Snort commands, it is critical to understand *what* an IDS/IPS does, *where* it sits, and *how* it detects threats.

### 1. Intrusion Detection System (IDS) — Passive Monitoring
An **IDS** inspects copies of traffic and generates **alerts** when it spots malicious patterns, anomalies, or policy violations. It does **not** automatically block traffic—an analyst or external tool has to act on the alert.

- **NIDS (Network IDS):** Monitors traffic flowing across network segments / an entire subnet to detect signatures and alert.
- **HIDS (Host-based IDS):** Monitors traffic and activity on a **single endpoint host** and raises alerts if suspicious patterns appear.

### 2. Intrusion Prevention System (IPS) — Active Protection
An **IPS** sits inline with traffic and actively **blocks, drops, or terminates** suspicious connections as soon as a threat is identified.

- **NIPS (Network IPS):** Monitors traffic across the network/subnet and terminates connections when a known threat signature matches.
- **Behaviour-based IPS / NBA (Network Behaviour Analysis):** Monitors network traffic across the subnet and terminates connections when **anomalies** deviate from normal behavior.
  - **Why NBA is powerful:** Unlike signature-only tools, NBA can catch **new or unknown threats**.
  - **Baselining (Training Period):** NBA must first learn what "normal" benign traffic looks like—this training phase is called **baselining**. If an attack happens *during* baselining, the system learns malicious traffic as normal; if training is incomplete, it triggers high false positives.
- **WIPS (Wireless IPS):** Monitors wireless network traffic specifically and terminates malicious wireless connections.
- **HIPS (Host-based IPS):** Protects a **single endpoint host** just like HIDS, except instead of only alerting, HIPS actively stops/terminates the malicious connection.

### 3. Detection & Prevention Techniques

| Technique | How It Works | Best Used For |
| :--- | :--- | :--- |
| **Signature-Based** | Matches traffic against explicit rules/patterns of known attacks. | Detecting **known threats** accurately with low false positives. |
| **Behaviour-Based** | Compares observed activity against a learned normal baseline. | Detecting **new / unknown threats** that evade signatures. |
| **Policy-Based** | Compares traffic/events against defined security policies and system configurations. | Detecting **policy violations**. |

### 4. Snort Overview & Three Core Modes
Snort combines signature/rule matching with flexible operational modes:
- **Capabilities:** Live traffic analysis, attack and probe detection, packet logging, protocol analysis, real-time alerting, modular plugins, pre-processors, and cross-platform support (Linux & Windows).
- **Three Primary Modes:**
  1. **Sniffer Mode:** Reads packets from the network and displays them in the console (similar to `tcpdump`).
  2. **Packet Logger Mode:** Saves inbound and outbound IP packets to disk for later analysis.
  3. **NIDS / NIPS Mode:** Evaluates packets against rules to alert/log (IDS) or drop/reject malicious traffic (IPS).

### Questions & Answers

| Question | Explanation | Answer |
| :--- | :--- | :--- |
| Which IDS or IPS type can help you stop the threats on a local machine? | "Stop" = IPS (active prevention); "local machine" = Host-based. | `HIPS` |
| Which IDS or IPS type can help you detect threats on a local network? | "Detect" = IDS (passive alerting); "local network" = Network-based. | `NIDS` |
| Which IDS or IPS type can help you detect the threats on a local machine? | "Detect" = IDS; "local machine" = Host-based. | `HIDS` |
| Which IDS or IPS type can help you stop the threats on a local network? | "Stop" = IPS; "local network" = Network-based. | `NIPS` |
| Which described solution works by detecting anomalies in the network? | Network Behaviour Analysis (`NBA`) detects deviations/anomalies from normal network baselines. | `NBA` |
| According to the official description of the snort, what kind of NIPS is it? | The official quote states Snort *"can be used as a full-blown network intrusion prevention system."* | `full-blown` |
| NBA training period is also known as ... | Learning normal traffic before enforcing anomaly detection is called baselining. | `baselining` |

---

## Task 4: First Interaction with Snort

### What We Did & Why Each Flag Matters

Before capturing packets or enforcing rules, we verify Snort's installation and test its configuration file.

| Flag | Meaning | Why It Is Useful |
| :--- | :--- | :--- |
| `-V` / `--version` | Version info | Displays Snort version, **Build number**, and compiled library versions (`libpcap`, `PCRE`, `ZLIB`). Essential for verifying compatibility with rulesets. |
| `-c <file>` | Configuration file | Points Snort to a specific config file (e.g., `/etc/snort/snort.conf`). You can maintain multiple config files for different profiles, though only one is active per Snort run. |
| `-T` | Self-test mode | Validates the configuration, preprocessors, output plugins, and rule syntax **without** starting packet capture, and prints the exact count of loaded rules. Always run `-T` before deploying config edits. |
| `-q` | Quiet mode | Suppresses the ASCII Snort pig banner and initialization status messages—keeps console output clean when scripting or reading alerts. |

#### Commands Used
```bash
# 1. Check Snort version and build number
snort -V

# 2. Validate the primary configuration file and count loaded rules
sudo snort -c /etc/snort/snort.conf -T

# 3. Validate the secondary configuration file and compare loaded rules
sudo snort -c /etc/snort/snortv2.conf -T
```

> **Why did we use `sudo` with `-T`?** Configuration files often reference restricted paths and rule directories; running with elevated privileges ensures Snort can read every included rule file accurately.

### Questions & Answers

| Question | How We Got It | Answer |
| :--- | :--- | :--- |
| Run the Snort instance and check the build number. | Ran `snort -V` and checked `Version 2.9.7.0 GRE (Build 149)`. | `149` |
| Test the current instance with `/etc/snort/snort.conf` file and check how many rules are loaded with the current build. | Ran `sudo snort -c /etc/snort/snort.conf -T` and read the rule-loading summary. | `4151` |
| Test the current instance with `/etc/snort/snortv2.conf` file and check how many rules are loaded with the current build. | Ran `sudo snort -c /etc/snort/snortv2.conf -T` and checked how many rules were enabled in that config. | `1` |

---

## Task 5: Operation Mode 1 — Sniffer Mode

### What Sniffer Mode Does & Why We Use It
In **Sniffer Mode**, Snort behaves like `tcpdump`—it listens on a network interface and prints packet details directly to the terminal without requiring a configuration file or rule engine. This is useful for quick live network troubleshooting, confirming traffic actually hits an interface, and inspecting raw headers or payloads on the fly.

### Sniffer Mode Flags Breakdown

| Flag | What It Shows | Why & When to Use It |
| :--- | :--- | :--- |
| `-i <iface>` | Selects the network interface (e.g. `eth0`) | Essential on multi-interface hosts (routers, dual-NIC servers, IPS boxes) so Snort listens on the right segment. If there is only one interface, Snort uses it automatically. |
| `-v` | Verbose: IP & TCP/UDP/ICMP header details | Best when you want lightweight visibility (timestamps, source/dest IP & port, TTL, TOS, IP ID, packet length) without flooding your screen with payload bytes. |
| `-d` | Packet data (payload) + verbose info | Adds the application-layer payload in hex and ASCII (e.g., seeing queried domains in DNS or HTTP requests). |
| `-e` | Link-layer (Ethernet) headers | Adds Layer 2 details (source/destination MAC addresses, EtherType `0x800`, frame length). Useful when debugging ARP, switching, or MAC-level spoofing. |
| `-X` | Full packet dump in HEX & ASCII | Displays **every byte** from Ethernet frame through payload (`0x0000: ...`). Maximum detail for deep packet inspection. |

#### Example Combinations
```bash
sudo snort -v -i eth0    # Listen on eth0 with verbose IP/transport output
sudo snort -v            # Verbose mode on default interface
sudo snort -d            # Show headers + application payload
sudo snort -de           # Show link-layer headers + IP/transport + payload (same as: snort -d -e)
sudo snort -X            # Full raw hex + ASCII dump from Layer 2 to payload
```

> **Note:** Flags can be combined (`-vd`, `-de`) or written separately (`-v -d -e`). Pressing `CTRL+C` cleanly stops Snort and outputs capture statistics.

### Questions & Answers

| Question | Answer |
| :--- | :--- |
| You can practice the parameter combinations by using the `traffic-generator` script. | `No answer needed` |

---

## Task 6: Operation Mode 2 — Packet Logger Mode

### What Packet Logger Mode Does & Why We Use It
Viewing packets live on a busy terminal scrolls away quickly. **Packet Logger Mode** tells Snort to save captured packets to disk so you can preserve evidence, share captures, or analyze traffic offline using Snort, `tcpdump`, or Wireshark.

### Logger Mode Flags Breakdown

| Flag | Function | Why & When It Helps |
| :--- | :--- | :--- |
| `-l <dir>` | Sets log/alert output directory (default: `/var/log/snort`) | Passing `-l .` saves logs directly in your **current working directory**, keeping each incident or exercise neatly organized. |
| *(default binary)* | Writes a single binary `snort.log.<timestamp>` file | Fast to write, compact, and compatible with Snort `-r`, `tcpdump`, and Wireshark—best for full-fidelity packet capture. Cannot be read directly in a text editor. |
| `-K ASCII` | Writes human-readable ASCII logs grouped into folders by IP | Creates directories named after IP addresses containing protocol/port files (e.g., `UDP:3009-53`, `ICMP_ECHO`). Easy to inspect with `cat` or a text editor, **but** Snort `-r` cannot replay ASCII logs. |
| `-r <logfile>` | Reads a binary log/PCAP file back into Snort | Lets you filter and inspect saved binary captures offline. Supports **Berkeley Packet Filters (BPF)** like `icmp`, `tcp`, `'udp and port 53'`, or `'tcp port 80'`. |
| `-n <count>` | Stops after processing `N` packets | Prevents flooding the terminal when you only need to inspect the first few packets (e.g., `-n 10`). |

### Crucial Linux Permission Nuance
Because Snort needs `sudo` (root) privileges to sniff live interfaces, the generated log files and folders are owned by `root`. To read them without `sudo`, either:
1. Run inspection commands with `sudo`, **or**
2. Change ownership: `sudo chown <username> -R <directory>` (where `-R` applies ownership recursively).

### Walkthrough of Commands & Solutions

1. **Capturing in ASCII Mode (`TASK-6 Exercise`):**
   ```bash
   sudo snort -dev -K ASCII -l .
   # In a second terminal:
   sudo ./traffic-generator.sh   # Choose TASK-6 Exercise
   ```
   - **What happened & why:** Because we passed `-K ASCII`, Snort sorted all captured sessions into directories named after the involved IP addresses. Inside folder `145.254.160.237`, the filename format `<PROTO>:<src_port>-<dst_port>` immediately revealed the DNS connection file showing source port `3009` talking to port `53`.

2. **Inspecting the Binary Capture (`snort.log.1640048004`):**
   ```bash
   # Read the first 10 packets (check packet #10 IP ID and packet #8 Ack)
   sudo snort -r snort.log.1640048004 -n 10

   # Include payload (-d) on the first 4 packets to read HTTP headers (Referer on packet #4)
   sudo snort -dr snort.log.1640048004 -n 4

   # Filter only TCP port 80 packets using a BPF filter to get the exact packet count
   sudo snort -r snort.log.1640048004 'tcp port 80'
   ```
   - **Why `-d` was needed for Question 3:** `-r` alone displays headers; adding `-d` dumps the application-layer payload so we can read the HTTP `Referer:` header in packet 4.
   - **Why BPF `'tcp port 80'` worked for Question 5:** Passing a BPF expression after the `-r` file makes Snort process only matching packets and print the exact count (`41`) in its end-of-run statistics.

### Questions & Answers

| Question | Answer |
| :--- | :--- |
| Navigate to folder `145.254.160.237`. What is the source port used to connect port 53? | `3009` |
| Read the `snort.log.1640048004` file with Snort; what is the IP ID of the 10th packet? | `49313` |
| Read the `snort.log.1640048004` file with Snort; what is the referer of the 4th packet? | `http://www.ethereal.com/development.html` |
| Read the `snort.log.1640048004` file with Snort; what is the Ack number of the 8th packet? | `0x38AFFFF3` |
| Read the `snort.log.1640048004` file with Snort; what is the number of the `"TCP port 80"` packets? | `41` |

---

## Task 7: Operation Mode 3 — IDS/IPS Mode

### What IDS/IPS Mode Does & Why We Use It
Sniffing and logging record everything blindly; **IDS/IPS Mode** applies detection rules (via `-c <snort.conf>` or directly `-c <rules_file>`) so Snort actively identifies—and in IPS mode, blocks—malicious or interesting traffic.

### IDS/IPS Flags & Alert Modes

| Parameter | Purpose | Practical Benefit |
| :--- | :--- | :--- |
| `-c <conf> -T` | Test configuration | Catches typos/broken rule references before launching live monitoring. |
| `-N` | Disable logging | Turns off writing packet logs to disk while keeping console output/alerts active—great for quick tests without consuming disk space. |
| `-D` | Background (Daemon) mode | Runs Snort silently as a background service for automated/continuous monitoring (`ps -ef \| grep snort` to check PID; `sudo kill -9 <PID>` to stop). Only use once rules and config are stable. |
| `-A full` | Full alert mode *(default)* | Writes complete alert details (classification, priority, full headers) to the `alert` file; produces **no** console alerts. |
| `-A fast` | Fast alert mode | Writes compact one-line alert summaries (timestamp, message, IPs, ports) to the `alert` file; produces **no** console alerts. Lower disk/CPU overhead in busy networks. |
| `-A console` | Console alert mode | Prints `fast`-style alerts directly to the terminal—best for interactive rule testing and live demos. |
| `-A cmg` | CMG alert mode | Prints alert info **plus** packet headers and hex/ASCII payload to the terminal—best when debugging *why* a rule matched payload data. |
| `-A none` | Disable alerting | Suppresses the `alert` file completely while still creating binary packet log files (unless `-N` is also given). |
| `-c <local.rules>` | Run rules without `snort.conf` | Useful for fast isolated testing of custom rules, though slower and without full preprocessor setup. |
| `-Q --daq afpacket -i eth0:eth1` | Inline **IPS** mode | `-Q` enables inline mode; `--daq afpacket` bridges two interfaces (`eth0:eth1`) so rules using `drop`/`reject` physically block packets (`[Drop]`). |

### Walkthrough of the Exercise
```bash
sudo snort -c /etc/snort/snort.conf -A full -l .
# Run traffic-generator.sh -> choose "TASK-7 Exercise"
```
- **What happened & why:** Snort loaded `/etc/snort/snort.conf` (including its preprocessors and rulesets), saved full alerts into `./alert` (`-A full -l .`), and upon `CTRL+C` reported the detection summary. Inspecting the generated `alert` file showed **2** alerts corresponding to HTTP `GET` method detection.

### Questions & Answers

| Question | Answer |
| :--- | :--- |
| What is the number of the detected HTTP GET methods? | `2` |
| You can practice the rest of the parameters by using the `traffic-generator` script. | `No answer needed` |

---

## Task 8: Operation Mode 4 — PCAP Investigation

### What PCAP Investigation Does & Why We Use It
In incident response and threat hunting, analysts often receive `.pcap` captures after an event occurred. Snort's **PCAP Investigation Mode** feeds saved `.pcap` files straight through the IDS detection engine so you immediately see which rules fire, along with detailed protocol and preprocessor statistics—much faster than manually clicking through thousands of packets in Wireshark.

### PCAP Investigation Parameters

| Flag | Function | Why It Helps |
| :--- | :--- | :--- |
| `-r <file>` / `--pcap-single=<file>` | Read a single PCAP | Evaluates one capture against your ruleset and config. |
| `--pcap-list="a.pcap b.pcap"` | Read multiple PCAPs in one command | Batch-analyzes multiple captures in one run instead of repeating commands manually. |
| `--pcap-show` | Print PCAP filename during processing | Essential alongside `--pcap-list`—otherwise alerts from all files merge together and you cannot tell which PCAP generated which alert! |

### Commands Executed & Why Statistics Differ

1. **Inspecting `mx-1.pcap` with the default config (`/etc/snort/snort.conf`):**
   ```bash
   sudo snort -c /etc/snort/snort.conf -A full -l . -r mx-1.pcap
   ```
   - Snort outputs alerts into `./alert` AND prints a detailed end-of-run breakdown covering:
     - **Action Stats** (total generated alerts: `170`)
     - **Stream5 / TCP Preprocessor Stats** (`TCP Segments Queued`: `18`)
     - **HTTP Inspect Preprocessor Stats** (`HTTP response headers extracted`: `3`)

2. **Inspecting `mx-1.pcap` with the second config (`/etc/snort/snortv2.conf`):**
   ```bash
   sudo snort -c /etc/snort/snortv2.conf -A full -l . -r mx-1.pcap
   ```
   - Generated alerts dropped to `68` on the exact same PCAP—demonstrating how changing the config file (`-c`) changes which rulesets and preprocessors evaluate the traffic.

3. **Inspecting `mx-2.pcap` alone, then `mx-2.pcap` + `mx-3.pcap` together:**
   ```bash
   sudo snort -c /etc/snort/snort.conf -A full -l . -r mx-2.pcap
   sudo snort -c /etc/snort/snort.conf -A full -l . --pcap-list="mx-2.pcap mx-3.pcap"
   ```
   - Notice: `mx-2.pcap` generated `340` alerts (with `82` TCP packets in *Breakdown by protocol*), and combining `mx-2.pcap` + `mx-3.pcap` generated `1020` alerts total.

### Questions & Answers

| Question | Answer |
| :--- | :--- |
| `mx-1.pcap` with `snort.conf` — Number of generated alerts? | `170` |
| `mx-1.pcap` with `snort.conf` — How many TCP Segments are Queued? | `18` |
| `mx-1.pcap` with `snort.conf` — How many "HTTP response headers" were extracted? | `3` |
| `mx-1.pcap` with `snortv2.conf` — Number of generated alerts? | `68` |
| `mx-2.pcap` with `snort.conf` — Number of generated alerts? | `340` |
| `mx-2.pcap` with `snort.conf` — Number of detected TCP packets? | `82` |
| `mx-2.pcap` and `mx-3.pcap` with `snort.conf` — Number of generated alerts? | `1020` |

---

## Task 9: Snort Rule Structure

Writing effective Snort rules is the heart of network intrusion detection. Every rule has two parts:
1. **Rule Header** *(mandatory)*: Defines **who/where/what protocol** and **what action** to take.
2. **Rule Options** *(technically optional syntax-wise, but essential in practice)*: Enclosed in `(...)` and separated by `;`, defining **deep inspection criteria and alert metadata**.

```text
[Action] [Protocol] [Src IP] [Src Port] [Direction] [Dst IP] [Dst Port] ( [Options]; )
```

### Part 1: Rule Header Breakdown

#### 1. Actions
| Action | Behavior | When to Use |
| :--- | :--- | :--- |
| `alert` | Generate an alert and log the packet | Standard IDS monitoring. |
| `log` | Log the packet without alerting | Silent forensic recording. |
| `drop` | Block and log the packet | Active IPS mode (`-Q`) when you want silent dropping. |
| `reject` | Block, log, and actively terminate the session (TCP RST / ICMP error) | Active IPS mode when you want immediate connection teardown. |

#### 2. Protocols
Snort 2 rule headers only support **4 protocols**: `ip`, `tcp`, `udp`, and `icmp`.
- **How do we detect application protocols like FTP or HTTP?** By specifying `tcp` in the header, filtering the relevant port (`21`, `80`), and inspecting payload strings with options like `content`.

#### 3. IP Addresses & Port Filtering
- **Single IP:** `192.168.1.56`
- **Subnet (CIDR):** `192.168.1.0/24`
- **List of IPs/Ports:** `[192.168.1.0/24, 10.1.1.0/24]` or `[21,23]`
- **Negation (`!`):** `!192.168.1.0/24` (exclude subnet) or `!21` (everything except port 21)
- **Port Ranges:**
  - `1:1024` — ports 1 through 1024
  - `:1024` — ports 0 through 1024
  - `1025:` — ports 1025 and above

#### 4. Direction Operators
- `->` : Source (left) to Destination (right) only.
- `<>` : Bidirectional (matches traffic traveling either way).
- *(Note: There is **no** `<-` operator in Snort!)*

---

### Part 2: Rule Options Breakdown

#### A. General Rule Options (Metadata & Management)
| Option | Purpose | Why It Matters |
| :--- | :--- | :--- |
| `msg:"...";` | Short human-readable event label | Printed inside alerts/console logs so analysts immediately know what triggered. |
| `sid:<id>;` | Unique Snort Rule ID | `<100` reserved; `100–999,999` built-in rules; `>=1,000,000` user-created local rules. Never duplicate SIDs across active rules! *(Note: one sentence in the room notes mentions `100.000.000` as a typo, while its scope list specifies `>=1,000,000`.)* |
| `reference:<type>,<id>;` | External link/CVE (e.g., `reference:cve,CVE-XXXX;`) | Speeds up incident investigation by pointing directly to vulnerability/threat context. |
| `rev:<num>;` | Revision number (`rev:1;`, `rev:2;`) | Tracks how many times a rule has been modified—Snort doesn't auto-version rules, so analysts must update `rev` whenever editing an existing rule. |

#### B. Payload Detection Options (Inspecting Packet Data)
| Option | Purpose | Why It Matters |
| :--- | :--- | :--- |
| `content:"...";` | Matches ASCII (`"GET"`) or HEX (`"\|47 45 54\|"`) bytes in payload | Detects specific strings, protocol commands, or malware signatures. Case-sensitive by default; multiple `content` checks increase precision at some CPU cost. |
| `nocase;` | Disables case sensitivity for the preceding `content` | Prevents trivial bypasses (e.g., `get` vs `GET`). |
| `fast_pattern;` | Prioritizes a specific `content` value for initial pattern matching | By default Snort uses the longest `content` string first; `fast_pattern` forces Snort to pre-filter on a faster/more unique match (always case-insensitive, once per rule). |

#### C. Non-Payload Detection Options (Inspecting Header Fields)
| Option | Purpose | Why It Matters |
| :--- | :--- | :--- |
| `id:<number>;` | Matches the **IP ID** header field | Useful for spotting crafted packets, specific scanning tools, or packet tracking. |
| `flags:<flags>;` | Matches TCP control flags (`F`=FIN, `S`=SYN, `R`=RST, `P`=PSH, `A`=ACK, `U`=URG) | Detects scan types (SYN scans, XMAS/NULL scans) or specific TCP session states (`PA` for Push-Ack). |
| `dsize:<range>;` | Filters payload byte size (`min<>max`, `>100`, `<100`) | Detects oversized buffers, covert channels, or tiny abnormal probes. |
| `sameip;` | Matches packets where Source IP == Destination IP | Catches classic Land DoS attacks and forged loopback/spoofed packets. |

---

### Practical Exercise Walkthrough (`task9.pcap`)

Base command used in `Task-Exercises/Exercise-Files/TASK-9`:
```bash
snort -c local.rules -A full -l . -r task9.pcap
```
> **Important Discipline:** Between each sub-question, delete/clear the previous `alert` file and comment out (`#`) earlier rules in `local.rules` so previous matches don't pollute the next rule's alert count!

1. **Question 1 — Filter IP ID `35369`:**
   - **Rule written:**
     ```snort
     alert ip any any <> any any (msg:"IP ID 35369 Detected"; id:35369; sid:1000001; rev:1;)
     ```
   - **Why `ip`?** An IP Identification (`id`) field belongs to the Layer 3 IP header regardless of whether the inner packet is TCP, UDP, or ICMP. Reading the triggered packet/log revealed an ICMP **Timestamp Request**.
   - **Answer:** `TIMESTAMP REQUEST`

2. **Question 2 — Filter packets with `SYN` flag:**
   - **Rule written:**
     ```snort
     alert tcp any any <> any any (msg:"TCP SYN Flag"; flags:S; sid:1000002; rev:1;)
     ```
   - **Why `tcp` + `flags:S`?** TCP flags exist only in TCP headers, and `S` matches packets with the SYN flag set.
   - **Answer:** `1`

3. **Question 3 — Filter packets with `Push-Ack` flags:**
   - **Rule written:**
     ```snort
     alert tcp any any <> any any (msg:"TCP PSH-ACK Flags"; flags:PA; sid:1000003; rev:1;)
     ```
   - **Why `flags:PA`?** Combining `P` (PSH) and `A` (ACK) matches packets carrying pushed application data/acknowledgements.
   - **Answer:** `216`

4. **Question 4 — Filter `UDP` packets with the same source & destination IP:**
   - **Rule written:**
     ```snort
     alert udp any any <> any any (msg:"UDP Same Source and Dest IP"; sameip; sid:1000004; rev:1;)
     ```
   - **Why `udp` + `sameip`?** Restricts protocol to `udp` while `sameip` checks whether `src_ip == dst_ip`.
   - **Answer:** `7`

5. **Question 5 — Modifying an existing rule:**
   - **Concept:** Whenever an analyst edits an existing rule (improving accuracy or performance), the **`rev`** option must be incremented to document the new revision.
   - **Answer:** `rev`

---

## Task 10: Snort2 Operation Logic — Points to Remember

Understanding what happens *inside* Snort helps diagnose misconfigurations and false negatives.

### 1. Snort's 5 Core Pipeline Components
1. **Packet Decoder:** Collects raw packets from the DAQ/interface and decodes Layer 2–4 protocols for the next stage.
2. **Pre-processors:** Normalize, defragment, and reassemble packets (e.g., TCP stream reassembly, HTTP decoding) *before* rules run—preventing attackers from evading rules via packet fragmentation or encoding tricks.
3. **Detection Engine:** Evaluates normalized packets against all active rules.
4. **Logging and Alerting:** Writes triggered alerts and captured packet logs.
5. **Outputs and Plugins:** Integrates with external backends (syslog, MySQL, etc.) and loads extra detection/management plugins.

### 2. Snort Ruleset Tiers
- **Community Rules:** Free under GPLv2, publicly accessible with no registration needed.
- **Registered Rules:** Free with an account; receives subscriber rules after a **30-day delay**.
- **Subscriber Rules (Paid):** Commercial Cisco Talos ruleset updated twice a week (Tuesdays & Thursdays) for immediate threat coverage.

### 3. Essential Sections in `/etc/snort/snort.conf`

#### Step #1: Network Variables
- `HOME_NET`: Protected internal subnet (e.g., `192.168.1.1/24` or `any`).
- `EXTERNAL_NET`: Untrusted outside network (commonly `!$HOME_NET` or `any`).
- `RULE_PATH` (`/etc/snort/rules`), `SO_RULE_PATH` (`$RULE_PATH/so_rules`), `PREPROC_RULE_PATH` (`$RULE_PATH/plugin_rules`).

#### Step #2: Decoder & Inline IPS Configuration (`DAQ`)
- **Data Acquisition Modules (DAQ)** abstract packet I/O so Snort can run passively or inline:
  1. `PCAP` — Default passive sniffer/IDS mode.
  2. `Afpacket` — Popular Linux **inline IPS** mode (`config daq: afpacket`, `config daq_mode: inline`).
  3. `Ipq` — Legacy Linux Netfilter inline mode (replaced `snort_inline`).
  4. `Nfq` — Modern Linux Netfilter queue inline mode.
  5. `Ipfw` — OpenBSD/FreeBSD inline mode using divert sockets.
  6. `Dump` — Testing mode for inline/normalization verification.
- `#config logdir`: Default persistent log directory (`/var/logs/snort` / `/var/log/snort`).

#### Step #6: Output Plugins
Controls how alerts and logs are stored/forwarded beyond basic console output.

#### Step #7: Customising Your Ruleset
- `include $RULE_PATH/local.rules` loads your custom local rules.
- `#include $RULE_PATH/<rulename>` — remove the `#` comment prefix to activate additional rulesets.

> **Configuration Safety Rule:** Never blindly replace a working `snort.conf` file—edit carefully or use proper rule-management tools, and always validate with `-T`.

### Questions & Answers

| Question | Answer |
| :--- | :--- |
| Read the task above. | `No answer needed` |

---

## Task 11: Conclusion & Best Practices

### Key Takeaways for Real-World Snort Usage
1. **Build rules incrementally:** Add options one step at a time so syntax or logic issues are immediately obvious.
2. **Don't reinvent the wheel:** Reuse and refine proven rules whenever possible.
3. **Always back up configs** before making modifications.
4. **Comment out (`#`) instead of deleting** working rules you temporarily don't need.
5. **Test offline first** using `-T` and sample `.pcap` files before pushing changes to production.

### Questions & Answers

| Question | Answer |
| :--- | :--- |
| Read the task above. | `No answer needed` |

---

## Quick Snort Cheatsheet

### Common Commands
```bash
# Version & config validation
snort -V
sudo snort -c /etc/snort/snort.conf -T

# Sniffer Mode
sudo snort -v -i eth0       # Verbose headers on eth0
sudo snort -d               # Headers + payload
sudo snort -de              # Headers + payload + Ethernet (MAC)
sudo snort -X               # Full raw HEX + ASCII dump

# Packet Logger Mode
sudo snort -dev -l .                 # Binary log in current dir
sudo snort -dev -K ASCII -l .        # Human-readable ASCII logs by IP
sudo snort -r snort.log.XXXX -n 10   # Read first 10 packets from binary log
sudo snort -r snort.log.XXXX 'tcp port 80'  # Filter binary log with BPF

# IDS / PCAP / IPS Mode
sudo snort -c /etc/snort/snort.conf -A console          # Live IDS alerts to console
sudo snort -c /etc/snort/snort.conf -A full -l . -r file.pcap   # Single PCAP
sudo snort -c /etc/snort/snort.conf -q --pcap-list="a.pcap b.pcap" -A console --pcap-show
sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A console  # Inline IPS
```

### Rule Template
```snort
alert tcp any any -> $HOME_NET 80 (msg:"Suspicious GET"; content:"GET"; nocase; fast_pattern; sid:1000001; rev:1;)
```
