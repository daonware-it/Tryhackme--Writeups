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

---

<br>
© 2025 by daonware 
<br>Created: July 23, 2025
<br>Last updated: July 23, 2025

License: [Public Domain / CC0](https://creativecommons.org/publicdomain/zero/1.0/)