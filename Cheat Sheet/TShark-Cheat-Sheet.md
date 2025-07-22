### TShark Cheat Sheet - Compact & Clear

<br>

# ✨ **Basics of TShark**

TShark is the command-line version of Wireshark for analyzing network traffic.

<br>

# 🔹 **Essential Parameters**

| Parameter | Description |
|:--------: |:-----------:|
| `-i` | Interface to capture from (e.g. eth0, wlan0) |
| `-r` | Read from a `.pcap` file |
| `-w` | Write captured packets to file |
| `-f` | **Capture Filter** (before capture, BPF syntax) |
| `-Y` | **Display Filter** (after capture, Wireshark syntax) |
| `-V` | Verbose packet details |
| `-q` | Quiet mode (suppress output) |
| `-c` | Limit number of packets |
| `-a` | Auto-stop conditions (time, size, etc.) |
| `-b` | Ring buffer options |
| `-T` | Output format (e.g. `-T fields`) |
| `-e` | Define fields for output |
| `-E header=y` | Show column headers with fields output |

---

# 🔹 **Capture Filters (`-f`)**

Applied **before capture**. Efficient but limited. Based on BPF syntax.

### ✅ **Examples:**

| Command | Purpose |
|:-------:|:-------:|
| `-f "port 80"` | HTTP traffic |
| `-f "tcp"` | Only TCP |
| `-f "udp"` | Only UDP |
| `-f "host 192.168.1.5"` | In/Out traffic from/to IP |
| `-f "src host 10.0.0.10"` | Only incoming |
| `-f "dst host 8.8.8.8"` | Only outgoing |
| `-f "net 192.168.0.0/16"` | Entire subnet |
| `-f "tcp or udp"` | TCP and UDP |
| `-f "tcp and not port 22"` | TCP except SSH |
| `-f "portrange 80-100"` | Port range |
| `-f "ether host F8:DB:C5:A2:5D:81"` | Filter by MAC address |
| `-f "ip proto 1"` | ICMP protocol |

---

# 🔹 **Display Filters (`-Y`)**

### ✅ **Examples by Protocol:**

### 📡 **IP:**

| Filter | Purpose |
|:------:|:-------:|
| `ip.addr == 10.0.0.5` | IP appears anywhere |
| `ip.src == 10.0.0.5` | Only source IP |
| `ip.dst == 10.0.0.5` | Only destination IP |
| `ip.addr == 10.0.0.0/24` | Subnet range |

### 🛡️ **TCP:**

| Filter | Purpose |
|:------:|:-------:|
| `tcp.port == 80` | Any TCP on port 80 |
| `tcp.srcport == 80` | Source port 80 |
| `tcp.flags.syn == 1` | SYN flag set |
| `tcp.len > 0` | TCP with payload |

### 🌐 **HTTP:**

| Filter | Purpose |
|:------:|:-------:|
| `http` | All HTTP packets |
| `http.request.method == "GET"` | Only GET |
| `http.request.method matches "(GET|POST)"` | Regex for GET/POST |
| `http.server contains "Apache"` | Apache server detection |
| `http.user_agent contains "sqlmap"` | Suspicious user agents |
| `http.host contains "google.com"` | Hostname analysis |
| `http.host == ""` | Empty HTTP host (possibly manipulated) |

### 🌍 **DNS:**

| Filter | Purpose |
|:------:|:-------:|
| `dns` | All DNS packets |
| `dns.qry.name` | Query names |
| `dns.qry.type == 1` | A-records only |

### 📩 **POST Data:**

| Filter | Purpose |
|:------:|:-------:|
| `urlencoded-form.key` | POST field name |
| `urlencoded-form.value` | POST value |

### 🚀 **Others:**

| Filter | Purpose |
|:------:|:-------:|
| `icmp` | ICMP packets |
| `bootp.option.hostname` | DHCP hostname |
| `ipv6` | IPv6 traffic only |

---

# 🔹 **Filter Operators: `contains` & `matches`**

| Operator | Description | Example |
|:--------:|:-----------:|:--------:|
| `contains` | Case-sensitive substring | `'http.server contains "Apache"'` |
| `matches` | Regex (case-insensitive) | `'http.request.method matches "(GET|POST)"'` |

Note: Only works on strings, **not integers** (like port numbers).

---

# 🔹 **Combining Display Filters**

| Combination | Example |
|:----------:|:--------:|
| AND | `http and ip.src == 10.0.0.1` |
| OR | `http.request or http.response` |
| NOT | `not tcp.port == 22` |
| Complex | `(http.request.method == GET) and (ip.dst == 10.0.0.5)` |

---

# 📊 **Field Output (`-T fields`)**

| Parameter | Meaning |
|:--------:|:--------:|
| `-T fields` | Output in field format |
| `-e <field>` | Specify fields to show |
| `-E header=y` | Show column headers |

### Example:

```bash
tshark -r file.pcapng -T fields -e ip.src -e ip.dst -E header=y
```

---

# 🔹 **Export HTTP Files**

```bash
tshark -r capture.pcapng --export-objects http,EXPORT_FOLDER -q
```

---

# 🧪 **Use Cases for Security Analysts**

### 1. Extract DHCP Hostnames:

```bash
tshark -r hostname.pcapng -T fields -e dhcp.option.hostname | awk NF | sort -r | uniq -c | sort -r
```

### 2. Extract DNS Queries:

```bash
tshark -r dns-queries.pcap -T fields -e dns.qry.name | awk NF | sort -r | uniq -c | sort -r 
```

### 3. Extract User Agents:

```bash
tshark -r user-agents.pcap -T fields -e http.user_agent | awk NF | sort -r | uniq -c | sort -r
```

### 4. Apache Servers & HTTP Methods:

```bash
# Detect Apache servers
tshark -r demo.pcapng -Y 'http.server contains "Apache"' -T fields -e ip.src -e ip.dst -e http.server -E header=y

# Only GET/POST requests
tshark -r demo.pcapng -Y 'http.request.method matches "(GET|POST)"' -T fields -e ip.src -e ip.dst -e http.request.method -E header=y
```

---

# ⚙️ **Auto-Stop & Ring Buffer**

### `-a` Auto-Stop Options:

| Sub-Param | Description |
|:---------:|:-----------:|
| `duration:x` | Stop after X seconds |
| `filesize:x` | Stop after X KB |
| `packets:x` | Stop after X packets |

### Example:

```bash
tshark -i eth0 -a duration:60 -w capture.pcap
```

### `-b` Ring Buffer Options:

| Sub-Param | Description |
|:---------:|:-----------:|
| `files:X` | Number of rotating files |
| `filesize:X` | Max size in KB per file |
| `duration:X` | Max time per file |

### Example:

```bash
tshark -i eth0 -b filesize:1024 -b files:5 -w capture.pcap
```


<br>

---

<br>
© 2025 by daonware 
<br>Created: July 23, 2025
<br>Last updated: July 23, 2025

License: [Public Domain / CC0](https://creativecommons.org/publicdomain/zero/1.0/)
