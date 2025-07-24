# **Linux Forensics - Comprehensive Summary**

### **1. Introduction**

- Linux is an open-source operating system with many distributions (Ubuntu, RedHat, ArchLinux, Debian, etc.).
- It is modular, lightweight, and customizable.
- Found in servers, smartphones, embedded systems, and even cars.

---

### **2. System Identification & Basic Information**

#### 🧾 `/etc/os-release` - Operating System Info
- Use: `cat /etc/os-release`
- Provides details like OS name, version, and homepage.

<pre><code>NAME="Ubuntu"
VERSION="20.04.6 LTS (Focal Fossa)"
ID=ubuntu
PRETTY_NAME="Ubuntu 20.04.6 LTS"
VERSION_ID="20.04"
</code></pre>

---

#### 🧾 `/etc/passwd` - User Information
- Fields: `username:password:UID:GID:comment:home_directory:shell`
- Use: `cat /etc/passwd | column -t -s ':'`

<pre><code>root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
syslog:x:104:111::/home/syslog:/usr/sbin/nologin
ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
</code></pre>

> 🔒 Passwords are not stored here; only an `x` placeholder. The real password hashes are in `/etc/shadow`.

---

#### 🧾 `/etc/shadow` - Password Hashes (root only)
- Format: `username:password_hash:last_change:min:max:warn:inactive:expire`

<pre><code>root:$6$abcXYZ$skjl98Ubsdf...:18539:0:99999:7:::
daemon:*:18539:0:99999:7:::
syslog:*:18539:0:99999:7:::
ubuntu:$6$abcd1234$afdl34JFDJSlsdfjlk...:18539:0:99999:7:::
</code></pre>

> `$6$` = SHA-512 hash, `*` or `!` = no login possible

---

#### 🧾 `/etc/group` - Group Information
- Fields: `groupname:password:GID:members`

<pre><code>root:x:0:
daemon:x:1:
adm:x:4:syslog,ubuntu
sudo:x:27:ubuntu
ubuntu:x:1000:
www-data:x:33:
</code></pre>

---

#### 🧾 Sudo Privileges
- Configured in `/etc/sudoers` or `/etc/sudoers.d/`
- View with: `sudo cat /etc/sudoers`
- Sample line:

<pre><code>ubuntu  ALL=(ALL:ALL) ALL
</code></pre>

---

#### 🧾 Login History
- Successful logins: `/var/log/wtmp` → `last -f /var/log/wtmp`
- Failed attempts: `/var/log/btmp` → `lastb -f /var/log/btmp`

---

#### 🧾 Authentication Logs
- File: `/var/log/auth.log`
- Use: `cat /var/log/auth.log | grep 'sudo'`

---

### **3. System Configuration**

#### 🧾 Hostname
- File: `/etc/hostname`
- View with: `cat /etc/hostname`

<pre><code>ubuntu-machine
</code></pre>

---

#### 🧾 Timezone
- File: `/etc/timezone`
- View with: `cat /etc/timezone`

<pre><code>Europe/Berlin
</code></pre>

---

#### 🧾 Network Interfaces
- File: `/etc/network/interfaces`
- View with: `cat /etc/network/interfaces`

<pre><code>auto lo
iface lo inet loopback
</code></pre>

---

#### 🧾 IP and MAC Addresses (Live)
- Use: `ip address show`

---

#### 🧾 Active Network Connections
- Use: `netstat -natp` *(may require root)*

---

#### 🧾 Running Processes
- Use: `ps aux`

---

#### 🧾 DNS Configuration
- Hosts: `/etc/hosts`
- DNS resolvers: `/etc/resolv.conf`

<pre><code>127.0.0.1 localhost
8.8.8.8  google-public-dns-a.google.com
</code></pre>

---

### **4. Persistence Mechanisms**

#### 🧾 Cron Jobs
- System-wide: `/etc/crontab`
- View: `cat /etc/crontab`

<pre><code>*/5 * * * * root /usr/bin/some-script.sh
</code></pre>

---

#### 🧾 Startup Scripts
- Directory: `/etc/init.d/`
- List: `ls /etc/init.d/`

---

#### 🧾 Shell Startup Files
- Per user: `~/.bashrc`
- System-wide: `/etc/bash.bashrc`, `/etc/profile`

---

### **5. Execution Traces**

#### 🧾 Sudo Commands
- File: `/var/log/auth.log`
- Filter: `grep 'COMMAND=' /var/log/auth.log*`

---

#### 🧾 Bash History
- File: `~/.bash_history`
- Shows previously entered shell commands.

---

#### 🧾 Vim History
- File: `~/.viminfo`
- Stores past searches and opened files in Vim.

---

### **6. Important Log Files**

#### 🧾 Syslog
- File: `/var/log/syslog*`
- Example: `cat /var/log/syslog* | head`

---

#### 🧾 Authentication Log
- File: `/var/log/auth.log*`
- Contains sudo usage, login attempts, and user changes.

---

#### 🧾 Third-Party Logs
- Apache: `/var/log/apache2/`
- Samba: `/var/log/samba/`
- Other services may write logs to `/var/log/` as well.

---

> 🛠️ Forensic Tip: Combine `grep`, `cut`, and `awk` to extract useful data from logs.


<br>

---

<br>
© 2025 by daonware 
<br>Created: July 24, 2025
<br>Last updated: July 24, 2025

License: [Public Domain / CC0](https://creativecommons.org/publicdomain/zero/1.0/)