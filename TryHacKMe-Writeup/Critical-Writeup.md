
<div align="center">

| Page | Room name| Difficulty | estimated time | premium room? | author|
|:---|:---:|:---:|:---:|:---:|:---:|
|[Tryhackme.com](https://www.tryhackme.com)|[Critical](https://tryhackme.com/room/critical) |Easy|60 min|Free| [tryhackme](https://tryhackme.com/p/tryhackme) <br> [dplastico](https://tryhackme.com/p/dplastico) <br> [rePl4stic](https://tryhackme.com/p/rePl4stic) |

</div>

----

# **Table of contents**

- [**Table of contents**](#table-of-contents)
  - [**1. Incident Scenario**](#1-incident-scenario)
  - [**Learning Objectives**](#learning-objectives)
  - [**3. Prerequisites**](#3-prerequisites)
  - [**4. Key Concepts of Memory Forensics**](#4-key-concepts-of-memory-forensics)
  - [**5. Memory Acquisition: Environment and Tools**](#5-memory-acquisition-environment-and-tools)
    - [**5.1 Imaging Tools**](#51-imaging-tools)
    - [**5.2 Accessing the Machine**](#52-accessing-the-machine)
  - [**6. Gathering Information**](#6-gathering-information)
  - [**7. Identifying Suspicious Activities**](#7-identifying-suspicious-activities)
    - [**7.1 Suspicious Activity**](#71-suspicious-activity)
    - [**7.2 The Search Begins**](#72-the-search-begins)
  - [**8. Detailed Investigation of `update.exe`**](#8-detailed-investigation-of-updateexe)
    - [**8.1 Getting the "Loot"**](#81-getting-the-loot)



## **1. Incident Scenario**

Our user "Hattori" reported strange behavior on his computer. He noticed that several PDF files were encrypted, including a critical company document named `important_document.pdf`. Suspecting that credentials might have been stolen, the **DFIR team** (Digital Forensics and Incident Response) was called in and collected evidence. Join the team to investigate the incident and learn how to extract information from a memory dump in a practical scenario.

## **Learning Objectives**

In this room, you will learn:

- **Basic concepts** of memory forensics.
- How to **set up the environment and access** the target.
- **Gathering information** from the compromised system.
- **Searching for suspicious activities** using collected data.
- **Extracting and analyzing data** from memory.
- **Conclusion and next steps** after finishing the room.

You will learn how to extract key information and artifacts when performing memory forensics on a compromised **Windows operating system**. First, we cover basic concepts, then set up the environment, gather information, search for suspicious activities, and finally extract data to help identify potential malicious actors.

## **3. Prerequisites**

It is recommended to understand the following topics before starting:

- Analysis of **volatile memory**
- **Volatility** framework
- **IR** (Incident Response) challenges

Let’s get started!

## **4. Key Concepts of Memory Forensics**

Let’s summarize some important concepts in memory forensics that are useful for our scenario.

In cybersecurity, **memory forensics** is a branch of computer forensics focused on analyzing **volatile memory**, typically on a compromised machine. For Windows OS, this means **RAM**, which is wiped on every reboot or shutdown, making it a crucial first step in incident response. Unlike disk forensics, memory analysis provides insight into the execution flow and live state of the system, often unavailable in logs or disk artifacts.

Memory analysis can provide an **instant snapshot** of applications or attacker actions, which is vital for building an **event timeline**.

Memory forensics tasks are divided into two main phases:  
**Memory Acquisition** and **Memory Analysis**.

During **memory acquisition**, we copy the live memory to a file (a **dump**) for analysis, avoiding data loss from accidental reboots and preserving evidence.

Next, in the **memory analysis phase**, we analyze the acquired dump.

## **5. Memory Acquisition: Environment and Tools**

With this knowledge, we start the first phase: **memory acquisition**, and determine the environment and tools for analyzing the dump.

### **5.1 Imaging Tools**

There are several ways to acquire memory from the target machine;  
the choice depends on personal preference and the OS used for imaging.  
Some tools include:

- **Windows:** FTK Imager, WinPmem
- **Linux:** LIME
- **macOS:** osxpmem

In our scenario, **FTK Imager** was used to create a memory dump, which was then copied to a Linux machine for analysis.

### **5.2 Accessing the Machine**

Before proceeding, start the lab by clicking **"Start Machine"**. The VM will be accessible on the right side of the split screen. If not visible, use the blue **"Show Split View"** button. You can also connect via `SSH` with:

- **Username:** `analyst`
- **Password:** `forensics`
- **IP:** `MACHINE_IP`

A memory dump named `memdump.mem` is located in `/home/analyst`.

For analysis, we use **Volatility3**, a popular tool for incident response. An alias is set up so the command `vol` runs Volatility3 in the terminal. Use `-h` for help:

```
Volatility3
user@machine$ vol -h
usage: volatility [-h] [-c CONFIG] [--parallelism [{processes,threads,off}]] [-e EXTEND] [-p PLUGIN_DIRS] [-s SYMBOL_DIRS]
                  [-v] [-l LOG] [-o OUTPUT_DIR] [-q] [-r RENDERER] [-f FILE] [--write-config] [--save-config SAVE_CONFIG]
                  [--clear-cache] [--cache-path CACHE_PATH] [--offline] [--single-location SINGLE_LOCATION]
                  [--stackers [STACKERS [STACKERS ...]]]
                  [--single-swap-locations [SINGLE_SWAP_LOCATIONS [SINGLE_SWAP_LOCATIONS ...]]]
                  plugin ...
```

Volatility offers many options for memory analysis. We can list plugins for each OS; in our case, we analyze a Windows dump. List Windows plugins with `windows` and `--help`:

```
Volatility3
user@machine$ vol windows --help
usage: volatility [-h] ... plugin ...
volatility: error: argument plugin: plugin windows matches multiple plugins (windows.bigpools.BigPools, windows.cachedump.Cachedump, ... windows.pstree.PsTree, ...)
```

Plugins are extremely helpful for quickly searching a memory dump for specific data types. Here’s a summary of some important plugins:

| Plugin | Description |
|:---------|:---------|
| `windows.cmdline` | Lists command line arguments of processes |
| `windows.drivermodule` | Detects if loaded drivers are hidden by rootkits |
| `windows.filescan` | Scans for file objects present in the Windows memory image |
| `windows.getsids` | Shows SIDs (Security Identifiers) for each process |
| `windows.handles` | Lists open handles for processes |
| `windows.info` | Shows OS & kernel details of the memory sample |
| `windows.netscan` | Scans for network objects in the Windows memory image |
| `windows.netstat` | Traverses network tracking structures in the memory image |
| `windows.mftscan` | Scans for Alternate Data Streams (ADS) |
| `windows.pslist` | Lists processes present in the memory image |
| `windows.pstree` | Lists processes in a tree structure based on parent PID |

Now that we know how to access our environment and which tools to use, let’s move on to data analysis.

## **6. Gathering Information**

Getting information about the target is crucial for context and accurate analysis. This step helps us understand the architecture and OS, ensuring our results are relevant and legitimate.

We can get target info using the `-f` switch to specify the file (`memdump.mem`) and the `windows.info` plugin:

```
Volatility
user@machine$ vol -f memdump.mem windows.info
Volatility 3 Framework 2.5.2
Progress:  100.00PDB scanning finished                        
Variable                           Value

Kernel Base                        0xf9066161c000
DTB                                0x1ac000
Symbols                            file:///home/analyst/volatility3-2.5.2/volatility3/symbols/windows/ntkrnlmp.pdb/4DBE144182FF4156845CD3BD8B65
Is64Bit                            False
IsPAE                              False
layer_name                         0 WindowsIntel32e
memory_layer                       1 FileLayer
KdVersionBlock                     0xf8066222a400
Major/Minor                        15.19041
KeNumberProcessors                 2
SystemTime                         2024-02-24 22:52:52
NtSystemRoot                       C:\Windows
NtProductType                      NtProductWinNt
NtMajorVersion                     7
NtMinorVersion                     0
PE MajorOperatingSystemVersion     10
PE MinorOperatingSystemVersion     0
PE Machine                         34404
PE TimeDateStamp                   Sat Jan 13 03:45:32 2085
```

The output shows relevant info to identify the machine, such as *architecture*, *number of processors*, and *version*. This helps correlate data with other analyses on separate hardware or network devices.

## **7. Identifying Suspicious Activities**

With system info collected, we now look for suspicious activities in the memory dump.

### **7.1 Suspicious Activity**

Suspicious activities include technical anomalies such as **unexpected processes, unusual network connections, or registry modifications**. These often signal security threats like malware or data leaks.

### **7.2 The Search Begins**

We start by observing potential network activity. Use the `windows.netstat` plugin to check for interesting or unusual connections. Remote access or connections to suspicious websites are what we’re looking for.

Navigate to `/home/analyst` and run:

```
vol -f memdump.mem windows.netstat
```

From the output, we see a connection on port `3389` from IP `192.168.182.139` at timestamp `2024-02-24 22:47:52.00`; this could indicate **initial attacker access**.

Next, let’s look at processes. The Volatility plugin `windows.pstree` shows a tree of running processes:

```
vol -f memdump.mem windows.pstree

//Example
Volatility 3 Framework 2.5.2
Progress:  100.00          PDB scanning finished
Offset (V)          Name                    Pid   PPid  Threads   Handles  Session  Wow64  CreateTime                 
-------------------- -------------------- ------ ------ -------- --------- -------- ------ ---------------------------
0xffffa3028c24c080   System                 4      0       120     789       0      False  2025-07-25 09:00:10.000000
0xffffa3028c24c100   smss.exe             348    4        22     150       0      False  2025-07-25 09:00:11.000000
0xffffa3028c24c180     csrss.exe          600    348      40     400       1      False  2025-07-25 09:00:12.000000
0xffffa3028c24c200     winlogon.exe       680    348      25     300       1      False  2025-07-25 09:00:13.000000
0xffffa3028c24c280       services.exe       760    680      80     650       0      False  2025-07-25 09:00:15.000000
0xffffa3028c24c300         svchost.exe      880    760      50     450       0      False  2025-07-25 09:00:18.000000
0xffffa3028c24c380         svchost.exe     1020    760      60     500       0      False  2025-07-25 09:00:19.000000
0xffffa3028c24c400         lsass.exe       940    760      30     280       0      False  2025-07-25 09:00:20.000000
0xffffa3028c24c480   explorer.exe         1200   680      95     750       1      False  2025-07-25 09:01:05.000000
0xffffa3028c24c500     chrome.exe         2500   1200    180    1500       1      False  2025-07-25 09:05:30.000000
0xffffa3028c24c580     **update_utility.exe** 3120   1200     45     320       1      False  2025-07-25 10:15:22.000000  <-- Suspicious: child of explorer.exe
0xffffa3028c24c600       **winupdater.exe** 4560   3120     15     180       1      False  2025-07-25 10:15:40.000000  <-- Suspicious: child of update_utility.exe
0xffffa3028c24c680         **nc.exe** 5000   4560      5      50        1      False  2025-07-25 10:15:55.000000  <-- Highly suspicious: child of winupdater.exe, known Netcat process
0xffffa3028c24c700           cmd.exe        5500   5000      2      30        1      False  2025-07-25 10:16:10.000000
0xffffa3028c24c780             powershell.exe 6000   5500     10     100       1      False  2025-07-25 10:16:30.000000
0xffffa3028c24c800   notepad.exe          2800   1200     10      80        1      False  2025-07-25 09:30:10.000000
0xffffa3028c24c880   msteams.exe          3500   1200    100     800       1      False  2025-07-25 09:10:00.000000
```

The output shows a hierarchical view of running processes and their parent processes. For example, `Services.exe` is the parent of `dllhost.exe`.

How do we identify suspicious processes? One common method is checking the process name; threat actors often use misleading names. Check if the process is normally present. Here’s a table of common Windows processes:

| Process Name                | Description                                                                                                   |
|-----------------------------|---------------------------------------------------------------------------------------------------------------|
| `System`                    | Kernel-level process managing kernel functions and hardware communication.                                    |
| `System Interrupts`         | Handles hardware interrupts; high values may indicate hardware issues.                                       |
| `svchost.exe`               | Generic host for Windows services.                                                                           |
| `explorer.exe`              | Windows Explorer, manages desktop, taskbar, start menu, and file operations.                                 |
| `csrss.exe`                 | Client/Server Runtime Subsystem, supports console and GUI apps, manages threads and windows.                 |
| `winlogon.exe`              | Handles user logon/logoff, Ctrl+Alt+Delete, and lock screen.                                                 |
| `taskhostw.exe`             | Host for DLL-based services.                                                                                 |
| `services.exe`              | Service Control Manager, manages Windows services.                                                           |
| `lsass.exe`                 | Local Security Authority Subsystem Service, manages security policies and authentication.                    |
| `smss.exe`                  | Session Manager Subsystem, first process after boot, starts kernel services and drivers.                     |
| `spoolsv.exe`               | Print spooler, manages print jobs.                                                                           |
| `dwm.exe`                   | Desktop Window Manager, manages graphical effects and window animations.                                     |
| `RuntimeBroker.exe`         | Manages permissions for Windows apps.                                                                        |
| `audiodg.exe`               | Isolates audio processing from other system processes.                                                       |
| `sihost.exe`                | Shell Infrastructure Host, manages graphical elements like start menu and taskbar.                           |
| `wininit.exe`               | Starts system services and mounts drives at boot.                                                            |
| `SearchIndexer.exe`         | Windows Search Indexer, creates index for faster searches.                                                   |
| `WUDFHost.exe`              | User-Mode Driver Framework Host, handles custom device drivers.                                              |
| `TrustedInstaller.exe`      | Windows Module Installer, manages installation and updates.                                                  |
| `mdm.exe`                   | Mobile Device Management, manages mobile devices and synchronization.                                        |
| `conhost.exe`               | Console Window Host, supports command-line programs.                                                         |
| `fontdrvhost.exe`           | Usermode Font Driver Host, manages fonts.                                                                    |
| `msmpeng.exe`               | Windows Defender Antivirus Service.                                                                          |
| `mstsc.exe`                 | Remote Desktop Client.                                                                                       |
| `rundll32.exe`              | Executes functions from DLLs.                                                                                |
| `ctfmon.exe`                | CTF Loader, manages alternative input methods.                                                               |
| `CompatTelRunner.exe`       | Compatibility Telemetry Runner, collects telemetry data for Microsoft.                                       |
| `sedsvc.exe`                | Windows Remediation Service, fixes update issues.                                                            |
| `musnotification.exe`       | Modern Update Service, manages update notifications.                                                         |
| `SecHealthUI.exe`           | Provides Windows Security app UI.                                                                            |
| `LogonUI.exe`               | Manages lock and logon screens.                                                                              |
| `BackgroundTaskHost.exe`    | Manages background tasks for apps.                                                                           |
| `nvcontainer.exe`           | NVIDIA process for driver and graphics settings.                                                             |
| `appvclient.exe`            | Microsoft App-V Client, manages virtual applications.                                                        |
| `GameBarPresenceWriter.exe` | Part of Xbox Game Bar, manages game recordings and broadcasts.                                               |
| `dasHost.exe`               | Device Association Framework Provider Host, manages device communication.                                    |
| `wmiprvse.exe`              | WMI Provider Host, provides system-wide info for apps.                                                       |
| `sppsvc.exe`                | Software Protection Service, manages license checks and activation.                                          |
| `lsm.exe`                   | Local Session Manager, manages user logins and sessions.                                                     |
| `mmc.exe`                   | Microsoft Management Console, central interface for admins.                                                  |
| `inetinfo.exe`              | IIS Admin Process, manages web server services.                                                              |
| `wbemcomn.exe`              | WMI process, manages system info access.                                                                     |
| `unsecapp.exe`              | Handles WMI requests for app and remote system communication.                                                |
| `taskeng.exe`               | Task Scheduler Engine, manages scheduled tasks.                                                              |
| `pcalua.exe`                | Program Compatibility Assistant, monitors apps for compatibility issues.                                     |
| `wlanext.exe`               | Manages WLAN connections and configuration.                                                                  |
| `dllhost.exe`               | COM Surrogate, hosts COM objects for Windows and apps.                                                       |
| `msiexec.exe`               | Windows Installer, manages installation, repair, and removal of programs.                                    |
| `rdrservices.exe`           | Remote Desktop Services, manages remote desktop connections.                                                 |

Reviewing the output, we see a process named `critical_updat`, which does not appear to be a system process. Its parent is `update.exe`, also not a standard Windows process.

```
PID    PPID   Name             Address         Threads  Handles  Session  Wow64
3384   7960   conhost.exe      0xe50edab37080  4        -        1       False
1648   7960   critical_updat   0xe50ed9dc1080  5        -        1       False
1648   7960   updater.exe      0xe50edab53080  6        -        1       False
6460   3196   FTK Imager.exe   0xe50edad09080 19        -        1       False
```

This process looks suspicious. Note details like **timestamp, PID, PPID**, and **memory offset**.

## **8. Detailed Investigation of `update.exe`**

With the collected info, let’s investigate the `critical_updat` process, which has a child process named `updater`. Let’s check where it is stored on disk using the `windows.filescan` plugin. The output is large, so redirect it to a file:

```
vol -f memdump.mem windows.filescan > filescan_out
```

Inspect the file with `cat` and filter with `grep`:

```
cat filescan_out | grep updater
```

The output shows the file is stored at `\Users\user01\Documents\updater.exe` or `C:\Users\user01\Documents\updater.exe`.

For more details like access or modification time, use the `windows.mftscan.MFTScan` plugin, also redirecting output:

```
vol -f memdump.mem windows.mftscan.MFTScan > mftscan_out
```

Search for `updater.exe`:

```
cat mftscan_out | grep updater
```

The last four timestamps correspond to **creation, modification, update, and access times**; note these.

### **8.1 Getting the "Loot"**

Let’s get info about the process. We’ll use Volatility to dump and analyze the memory region belonging to `updater.exe`.

Use the `windows.memmap` plugin, specifying the output directory with `-o .`, the `--dump` option, and the PID (for `updater.exe`, **1612**):

```
vol -f memdump.mem -o . windows.memmap --dump --pid 1612
```

After running the command, you’ll have a `.dmp` file in your working directory.

Since the file contains non-printable characters, use the `strings` command for analysis. Search for keywords like `HTTP`, `key`, or anything that might lead to an artifact. You can also scroll through the output with `less`:

```
strings pid.1612.dmp | less
strings pid.1612.dmp | grep "HTTP"
```

We immediately identify a possible key and a domain from a URL accessed by the process. Searching for `.pdf` also reveals interaction with `important_document.pdf`.

We can conclude that `updater.exe` accessed the document `important_document.pdf` and at some point accessed a "key" at `http://key.critical-update.com/encKEY.txt`.

To search for the HTTP request in memory, use `grep` with `-B` and `-A` to show lines before and after the match:

```
strings pid.1612.dmp | grep -B 10 -A 10 "http://key.critical-update.com/encKEY.txt"
```

Scrolling up, we see the HTTP request:

```
</html>
@s1/0/_dk_http://critical-update.com http://critical-update.com http://key.critical-update.com/encKEY.txt
HTTP/1.0 200 OK
```

```
Date: Sat, 24 Feb 2024 22:52:40 GMT
Content-type: text/plain
Content-Length: 9
Last-Modified: Fri, 23 Feb 2024 22:56:51 GMT
192.168.182.128
cafebabe
ul1/0/_dk_https://microsoft.com https://microsoft.com https://edge.microsoft.com/ct_category_en&version=1.*.*&channel
```

From the above, we see the contents of `encKey.txt` at the end of the HTTP request, and data with value `cafebabe`. This could be the **key** used by the attacker to encrypt PDFs, which was not downloaded to disk.

Excellent. We have gathered valuable information from the memory dump, including the possible key used to encrypt the documents.

----

<div align="center">

© 2025 by daonware  
Created: July 25, 2025  
Last updated: July 25, 2025

</div>