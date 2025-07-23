# 🗂️ **Windows Forensics Cheat Sheet**

### **1. System Information & User Accounts**

| Information | Registry Path | Description |
|:---:|:---:|:---:|
| OS Version | `<code>HKLM\SOFTWARE\Microsoft\WindowsNT\CurrentVersion</code>` | Windows version, build, edition |
| Current ControlSet | `<code>HKLM\SYSTEM\Select\Current</code>` | Indicates which ControlSet is currently used |
| Last Known Good ControlSet | `<code>HKLM\SYSTEM\Select\LastKnownGood</code>` | Last saved stable ControlSet |
| ControlSets | `<code>HKLM\SYSTEM\ControlSet001 ControlSet002</code>` | System state and configuration data |
| Computer Name | `<code>HKLM\SYSTEM\CurrentControlSet\Control\ComputerName\ComputerName</code>` | Hostname of the system |
| Timezone | `<code>HKLM\SYSTEM\CurrentControlSet\Control\TimeZoneInformation</code>` | Local timezone (important for timeline analysis) |
| Network Interfaces (active) | `<code>HKLM\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\Interfaces</code>` | Active IP addresses, DHCP, DNS etc. |
| Past Networks | `<code>HKLM\SOFTWARE\Microsoft\WindowsNT\CurrentVersion\NetworkList\Signatures\Managed <br> ....Unmanaged</code>` | Previously connected networks, last connection time |
| Autostart Program (User) | `<code>NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Run <br> RunOnce</code>` | Programs that launch at user login |
| Autostart Program (System) | `<code>HKLM\SOFTWARE\Microsoft\CurrentVersion\Run <br> RunOnce <br> Policies\Explorer\Run</code>` | System-wide autostart programs |
| Services (Startup Type) | `<code>HKLM\SYSTEM\CurrentControlSet\Services</code>` | Startup type via value `<code>Start</code>` <br> (0x02 = automatic start) |
| User Accounts (SAM Hive) | `<code>HKLM\SAM\Domain\Account\Users</code>` | User info, login count, password info |

<br>

### **2. User Activity & File Usage**

| Information | Registry Path | Description |
|:---:|:---:|:---:|
| Recently Opened Files (Recent Docs) | `<code>NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs</code>` | List of recently used files by type |
| Office MRU (Recent Files) | `<code>NTUSER.DAT\Software\Microsoft\Office\VERSION <br> /UserMRU/LiveID_###\FileMRU</code>` | Recently opened Office files, including full paths |
| ShellBags (Folder Layouts) | `<code>USRCLASS.DAT\Local\Settings\Software\Microsoft\Windows\Shell\Bags <br> BagMRU <br> NTUSER.DAT\Software\Microsoft\Shell\BagMRUS <br> Bags</code>` | Info about folder views, recently opened folders |
| Open/Save Dialog (MRU) | `<code>NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\ComDlg32\OpenSavePIDlMRU <br> LastVisitedPidlMRU</code>` | Recently used open/save dialog locations |
| Windows Explorer Address/Search Bars | `<code>NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths <br> WordWheelQuery</code>` | Path entries and search queries |

<br>

### **3. Executed Programs & Activity**

| Information | Registry Path | Description |
|:---:|:---:|:---:|
| UserAssist | `<code>NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\UserAssist\{GUID}\Count</code>` | Programs run via Explorer <br> (execution time, count) |
| ShimCache (AppCompatCache) | `<code>SYSTEM\CurrentControlSet\Session Manager\AppCompatCache</code>` | List of executed programs including size & timestamps |
| AmCache | `<code>C:\Windows\appcompat\Programs\Amcache.hve <br> Amcache.hve\Root\File\{VolumeGUID}\</code>` | Executed programs, install/delete times, SHA1 hashes |
| BAM/DAM (Background & Desktop Activity) | `<code>SYSTEM\CurrentControlSet\Services\bam\UserSettings\{SID} <br> ....\dam\UserSettings\{SID}</code>` | Background & desktop activity (programs & times) |

<br>

### **4. USB and Removable Media**

| Information | Registry Path | Description |
|:---:|:---:|:---:|
| USB Devices (Vendor, Product, Version) | `<code>SYSTEM\CurrentControlSet\Enum\USBSTOR <br> SYSTEM\CurrentControlSet\Enum\USB</code>` | Identification of all connected USB devices |
| USB Device Connection Timestamps | `<code>SYSTEM\CurrentControlSet\Enum\Ven_Prod_Version\USBSerial#\Properties\{GUID}\0064</code>` (first connection) <br> `<code>....0066</code>` (last connection) <br> `<code>....0067</code>` (last removal) | Time of first/last connection and removal |
| USB Volume Name | `<code>SOFTWARE\Microsoft\Windows Portable Devices\Devices</code>` | Display name of removable devices |

<br>

### **5. Registry Hive Storage Locations**

| Hive | File Path | Description |
|:---:|:---:|:---:|
| DEFAULT | `C:\Windows\System32\Config\DEFAULT` | `HKEY_USERS\DEFAULT` |
| SAM | `C:\Windows\System32\Config\SAM` | `HKEY_LOCAL_MACHINE\SAM` |
| SECURITY | `C:\Windows\System32\Config\SECURITY` | `HKEY_LOCAL_MACHINE\Security` |
| SOFTWARE | `C:\Windows\System32\Config\SOFTWARE` | `HKEY_LOCAL_MACHINE\Software` |
| SYSTEM | `C:\Windows\System32\Config\SYSTEM` | `HKEY_LOCAL_MACHINE\System` |
| NTUSER.DAT | `C:\Users\*USERNAME*\NTUSER.DAT` | `HKEY_CURRENT_USER` (at login) |
| USRCLASS.DAT | `C:\Users\*USERNAME*\AppData\Local\Microsoft\Windows\USRCLASS.DAT` | `HKEY_CURRENT_USER\Software\Classes` |
| AmCache Hive | `C:\Windows\AppCompat\Programs\Amcache.hve` | AmCache |

<br>

### **6. Tools for Data Collection & Analysis**

| Tool | Function | Notes |
|:---:|:---:|:---:|
| **KAPE** | Live data acquisition and analysis (also extracts registry hives) | CLI and GUI |
| **Autopsy** | Live system or image analysis, file extraction | GUI |
| **FTK Imager** | Mount disk images, extract files, *Obtain Protected Files* (live system only) | GUI |
| **Registry Editor (regedit.exe)** | View registry live only | Cannot load extracted hives |
| **Registry Viewer** | View individual registry hives, GUI | Loads one hive at a time, no log support |
| **Registry Explorer** | View multiple hives incl. transaction logs, bookmarks for key paths | Recommended for forensic analysis |
| **RegRipper** | Extracts forensic-relevant data from hives, outputs text reports | CLI/GUI, no logs; best used after merge with Registry Explorer |
| **AppCompatCacheParser** | Parses ShimCache from SYSTEM hive and outputs CSV | CLI, part of Eric Zimmerman's tools |
| **EZViewer** | GUI viewer for AppCompatCacheParser output | GUI, part of Eric Zimmerman's tools |
| **ShellBag Explorer** | Parses and displays ShellBag data from USRCLASS.DAT and NTUSER.DAT | GUI, Eric Zimmerman's tool |


<br>

# Windows Recovery with Tools for Forensic

---

## Introduction  
In previous tasks, we covered Windows forensics focusing on the Windows Registry to extract forensic artifacts such as system and user information, opened files/folders, executed programs, and connected external devices. However, forensic artifacts also exist outside the Registry.  
This document covers:  
- Different Windows file systems  
- Locations of artifacts in the file system  
- Evidence of program and file/folder execution and usage of external devices  
- Basics of deleted file recovery  
- Useful tools from Eric Zimmerman and Autopsy for artifact analysis  

---

## File Systems in Windows

### Why File Systems?  
Storage devices (e.g., hard drives, USB sticks) are just collections of bits. File systems organize these bits into meaningful data using defined structures and rules.

---

### File Allocation Table (FAT)  
- Microsoft's standard file system since the 1970s, still widely used on USB sticks, SD cards, and digital cameras.  
- Creates a table indexing locations of file data.

**Key Data Structures:**  
- **Cluster:** Basic unit of storage holding file data (bits).  
- **Directory:** Stores file metadata (name, start cluster, length).  
- **File Allocation Table:** Linked list indicating cluster status and pointing to next clusters.

**Versions Overview:**  
| Attribute              | FAT12    | FAT16     | FAT32       |
|------------------------|----------|-----------|-------------|
| Addressable Bits       | 12       | 16        | 28          |
| Max Number of Clusters | 4,096    | 65,536    | 268,435,456 |
| Supported Cluster Size | 512B–8KB | 2KB–32KB  | 4KB–32KB    |
| Max Volume Size        | 32MB     | 2GB       | 2TB         |

*Note:* Windows limits FAT32 formatting to 32GB, but supports larger volumes formatted by other OS.

---

### exFAT  
- Developed as FAT32 successor for large files and volumes (e.g., high-res photos/videos).  
- Standard file system for SD cards > 32GB.  
- Supports cluster sizes from 4KB up to 32MB.  
- Max file and volume size: 128 Petabytes (PB).  
- Maximum ~2,796,202 files per directory.  
- Lower system overhead and more efficient than FAT.

---

### NTFS (New Technology File System)  
- Introduced in 1993 with Windows NT 3.1, standard since Windows XP.  
- Provides advanced features: security, reliability, data recovery, support for large files/volumes.

**Key Features:**  
- **Journaling:** Logs metadata changes in $LOGFILE, aids recovery after crashes.  
- **Access Controls:** Manages file ownership and permissions per user.  
- **Volume Shadow Copy:** Enables restoring previous file versions (important for ransomware recovery).  
- **Alternate Data Streams (ADS):** Files can contain multiple data streams (e.g., Zone Identifier for internet downloads).  
- **Master File Table (MFT):** Extensive database of all files and objects on the volume.

**Important MFT Files:**  
- **$MFT:** Contains entries of all files/folders and their storage locations.  
- **$LOGFILE:** Logs transactions for filesystem integrity.  
- **$UsnJrnl:** Change journal with all file modifications.

**Analysis Tool:**  
- **MFTECmd.exe** by Eric Zimmerman for MFT analysis (CLI & GUI).  
- Example command to analyze MFT and save CSV:  
  `MFTECmd.exe -f <path-to-$MFT-file> --csv <output-path>`

---

## Deleted Files & Data Recovery

### Basics  
- Deleting a file removes filesystem references, but data remains until overwritten (unallocated space).  
- Recovery involves identifying unchanged data in unallocated clusters.

### Disk Image  
- Bit-by-bit copy of a storage device (including metadata).  
- Enables forensic analysis without altering original evidence.

### Recovery with Autopsy  
- Autopsy is a GUI tool for forensic disk image analysis.  
- Workflow:  
  - Create new case  
  - Add disk image as data source  
  - Select modules like "Recent Activity"  
  - Identify deleted files (marked with X) and extract.

---

## Forensic Artifacts Indicating Program Execution

### Windows Prefetch Files  
- Location: `C:\Windows\Prefetch`  
- Extension: `.pf`  
- Contain info about last run programs, run counts, accessed files and device handles.  
- Analysis Tool: **PECmd.exe** by Eric Zimmerman.  
- Example commands:  
  - Single file: `PECmd.exe -f <prefetch-file> --csv <output-path>`  
  - Directory: `PECmd.exe -d <prefetch-directory> --csv <output-path>`

### Windows 10 Timeline  
- Stores recently used applications/files in an SQLite database.  
- Location:  
  `C:\Users\<username>\AppData\Local\ConnectedDevicesPlatform\{randomfolder}\ActivitiesCache.db`  
- Analysis Tool: **WxTCmd.exe** by Eric Zimmerman.  
- Example:  
  `WxTCmd.exe -f <ActivitiesCache.db> --csv <output-path>`

### Windows Jump Lists  
- Shows recently opened files per application (right-click on taskbar icon).  
- Location:  
  `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Recent\AutomaticDestinations`  
- Analysis Tool: **JLECmd.exe** by Eric Zimmerman.  
- Example:  
  `JLECmd.exe -d <jumplist-directory> --csv <output-path>`

---

## Shortcut Files (LNK)

- Windows creates shortcuts for locally or remotely opened files.  
- Locations:  
  - `C:\Users\<username>\AppData\Roaming\Microsoft\Windows\Recent\`  
  - `C:\Users\<username>\AppData\Roaming\Microsoft\Office\Recent\`  
- Contain paths, creation, and access times of original files.  
- Analysis Tool: **LECmd.exe** by Eric Zimmerman.  
- Example:  
  `LECmd.exe -f <lnk-file> --csv <output-path>`

---

## IE/Edge History

- Includes visited websites and opened local files with `file:///` prefix.  
- Location:  
  `C:\Users\<username>\AppData\Local\Microsoft\Windows\WebCache\WebCacheV*.dat`  
- Analysis with Autopsy:  
  - Add logical files as data source  
  - Point to “triage” folder  
  - Enable only “Recent Activity” module  
- Shows local files accessed via browser.

---

## Setupapi.dev.log for USB Devices

- Logs device setup when USB devices are connected.  
- Location:  
  `C:\Windows\inf\setupapi.dev.log`  
- Contains device ID, serial number, first and last connection times.  
- Useful for proving USB device connections.

---

# Eric Zimmerman Tools Overview

| Tool        | Purpose                            | Key Options / Examples                                    |
|-------------|----------------------------------|-----------------------------------------------------------|
| MFTECmd     | NTFS MFT analysis                 | `-f <MFT-file> --csv <output-folder>`                     |
| PECmd       | Prefetch files analysis           | `-f <prefetch-file> --csv <output-folder>`                |
| WxTCmd      | Windows 10 Timeline analysis      | `-f <ActivitiesCache.db> --csv <output-folder>`           |
| JLECmd      | Windows Jump Lists analysis       | `-d <jumplist-folder> --csv <output-folder>`               |
| LECmd       | Shortcut (.lnk) file analysis     | `-f <lnk-file> --csv <output-folder>`                      |

---

**Note:**  
Paths in examples typically relate to user folders or desktop paths in forensic VMs or investigated systems. Replace `<username>` with the actual user name.


<br>

---

<br>
© 2025 by daonware 
<br>Created: July 23, 2025
<br>Last updated: July 24, 2025

License: [Public Domain / CC0](https://creativecommons.org/publicdomain/zero/1.0/)