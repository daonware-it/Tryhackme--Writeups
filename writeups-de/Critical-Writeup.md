<div align="center">

| Seite | Raumname| Schwierigkeit | geschätze Zeit | Premiumzugang | Ersteller|
|:---|:---:|:---:|:---:|:---:|:---:|
|[Tryhackme.com](https://www.tryhackme.com)|[Critical](https://tryhackme.com/room/critical) |Einfach|60 min|Kostenlos| [tryhackme](https://tryhackme.com/p/tryhackme) <br> [dplastico](https://tryhackme.com/p/dplastico) <br> [rePl4stic](https://tryhackme.com/p/rePl4stic) |

</div>

---

# **Inhaltsverzeichnis**


- [**Inhaltsverzeichnis**](#inhaltsverzeichnis)
  - [**1. Vorfall-Szenario**](#1-vorfall-szenario)
  - [**Lernziele**](#lernziele)
  - [**3. Lernvoraussetzungen**](#3-lernvoraussetzungen)
  - [**4. Wichtige Konzepte der Speicherforensik**](#4-wichtige-konzepte-der-speicherforensik)
  - [**5. Speichererfassung: Umgebung und Tools**](#5-speichererfassung-umgebung-und-tools)
    - [**5.1 Imaging-Tools**](#51-imaging-tools)
    - [**5.2 Zugriff auf die Maschine**](#52-zugriff-auf-die-maschine)
  - [**6. Informationen sammeln**](#6-informationen-sammeln)
  - [**7. Verdächtige Aktivitäten identifizieren**](#7-verdächtige-aktivitäten-identifizieren)
    - [**7.1 Verdächtige Aktivität**](#71-verdächtige-aktivität)
    - [**7.2 Die Suche beginnt**](#72-die-suche-beginnt)
  - [**8. Detaillierte Untersuchung des Prozesses `update.exe`**](#8-detaillierte-untersuchung-des-prozesses-updateexe)
    - [**8.1 Die "Beute" erhalten**](#81-die-beute-erhalten)


## **1. Vorfall-Szenario**

Unser Nutzer "Hattori" hat seltsames Verhalten an seinem Computer gemeldet. Er stellte fest, dass einige PDF-Dateien verschlüsselt wurden, darunter auch ein für das Unternehmen 
kritisches Dokument namens `important_document.pdf`. Daraufhin meldete er den Vorfall. Da der Verdacht bestand, dass Anmeldeinformationen gestohlen worden sein könnte, wurde das **DFIR-Team** 
(Digitale Forensics and Incident Response) hinzugezogen, das einige Beweismittel gesichert hat. Treten Sie dem Team bei, um den Vorfall zu untersuchen und zu erfahren, wie man in einem 
praktischen Szenario Informationen aus einem Speicherabbild (Memory Dump) gewinnt.

## **Lernziele**

In diesem Raum werden die folgenden Lernziele behandelt:

- **Grundlegende Konzepte** der Speicherforensik.
- Wie man die Umgebung **einrichtet und darauf zugreift**.
- **Informationsbeschaffung** vom kompromittierten Ziel.
- **Suche nach verdächtigten Aktivitäten** anhand der gewonnenen Informationen.
- **Extrahieren und Analysieren von Daten** aus dem Speicher.
- **Fazit und weitere Schritte** nach Abschluss des Raums.

In diesem Raum lernen wir, wie man wichtige Informationen und Artefakte extrahiert, wenn man eine Aufgabe der Speicherforensik an einem komprommierten **Windows-Betriebssystem** durchführt. 
Zuerst werden wir grundlegende Konzepte der Speicherforensik lernen, bevor wir unsere Arbeitsumgebung einrichten. Als Nächstes werden wir lernen, wie man grundlegende Informationen über das 
kompromittierte Ziel erhält und wie man nach verdächtigen Aktivitäten sucht. Schließlich werden wir interessante Daten extrahieren, die uns helfen können, potenzielle böswillige Akteuere zu identifizieren.


## **3. Lernvoraussetzungen**

Ein Verständis der folgenden Themen wird vor Beginn des Raumes empfohlen:

- Analyse von **votalilem Speicher**
- **Volatility**
- **IR** (Incident Response) Schwierigkeiten und Herausforderungen

Lasst uns unsere Aufgabe beginnen!


## **4. Wichtige Konzepte der Speicherforensik**

Fassen wir nun einige wichtige Konzepte der Speicherforensik zusammen, die für uns bei der Bearbeitung der vorgestellten Szenarios nützlich sein könnten.


In der Cybersicherheit ist die **Speicherforensik** ein Teilbereich der Computerforensik, der den **volatilen Speicher** analysiert, überlicherweise auf eine kompromierte Maschine. 
Unter Windows OS entspricht dies dem **Random Access Memory (RAM)**, dessen Inhalt bei jedem Neustart oder Herunterfahren gelöscht wird, was es zu einer der üblichen anfänglichen Aufgaben während 
eines Vorfalls macht. Der Prozess unterscheidet sich von der Festplattenforensik-Analyse, da er nicht nur Information darüber liefert, was sich auf dem Zielcomputer befindet, sondern uns auch 
Informationen über den Ausführungsfluss auf einem System liefert, die in normalen Speichereinheiten oder Anwendungsprotokoll möglicherweise nicht vorhanden sind.



Diese Speicheranalyse kann uns einen **sofortigen Schnappschuss** einer Anwendung oder einen Zeitstempel der Aktion eines Angreifers liefern. Dies ist entscheidend, da durch Speicherforensik 
gesammelte Beweise bei der Erstellung einer **Ereignischronologie** von unschätzbaren Wert sein können.



Wir können die Aufgabe der Speicherforensik in zwei Hauptphasen unterteilen:  
**Speichererfassung (Memory Acquistion)** und **Speicheranalyse (Memory Analysis)**.


Währen der **Speichererfassungsphase** kopieren wir den Live-Speicher in eine Datei, die überlichweise als **Dump** bezeichnet wird, um die Analyse durchzufügen, ohne das 
Risiko einzugehen, die Daten durch ein versehentlichen Neustart auf dem kompromittierten System zu verlieren und bei Bedarf einen Nachweis der Analyse 
zu haben.


Als Nächstes werden wir während der **Speicheranalysephase** den erhaltenen Speicher-Dump der forensischen Datei analysieren.


## **5. Speichererfassung: Umgebung und Tools**

Mit dem Wissen aus unserer vorherigen Aufgabe können wir die erste Phase unsere Speicherforensik-Aufgabe, die **Speichererfassung**, angehen und die Umgebung sowie die Tools bestimmen, 
die wir zur Analyse des Speicher-Dumps verwenden werden.


### **5.1 Imaging-Tools**

Es gibt verschiedene Möglichkeiten, den Speicher von der Zielmaschine bei Bedarf zu erfassen;  
mehrere Tools können uns dabei helfen, aber welches verwendet wird, hängt von der persönlichen Präferenz und dem bei der Imaging-Aufgabe 
verwendeten Betriebssystem ab.  
Einige diese Tools sind:

- **Windows:** FTK Imager, WinPmem
- **Linux:** LIME
- **macOS:** osxpmem

In unseren Szenario wurde **FTK Imager** verwendet, um ein Speicher-Dump der kompromittierten Maschine zu erstellen, der dann zur Analyse auf die Linux-Maschine kopiert wurde.


### **5.2 Zugriff auf die Maschine**

Bevor wir fortfahren, starten Sie das Lab, indem Sie auf die Schaltfläche **"Start Machine"** klicken. Das Laden dauert etwa `2` Minuten. Die VM wird auf der rechten Seite des geteilten Bildschirms zugänglich sein. Falls die VM nicht sichtbar ist, verwenden Sie die blaue Schaltfläche **"Show Split View** oben auf der Seite. Sie können sich auch direkt über `SSH` mit den folgenden Informationen mit der Maschine verbinden:

- **Benutzername:** `analyst`
- **Password:** `forensics`
- **IP:** `MACHINE_IP`

Ein Speicher-Dump namens `memdump.mem` befindet sich im Home-Verzeichnis unter `/home/analyst`.


Um den Dump zu analysieren, werden wir **Volatility3** verwenden, eine beliebte Wahl unter Analysten zur Inspektion eines Speichers während eines Vorfalls. In diesem Fall wurde tdas Tool installiert und ein Alias zur Vereinfachung erstellen; die Eingabe des Befehls `vol` führt Volatility3 im Terminal aus. Der Schalter `-h` kann das Hilfemenüs anzeigen, wie unten gezeigt.

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

Wie wir der obigen Ausgabe entnehmen können, bietet uns Volatility mehrere Optionen zur Analyse des Speichers. Wir können die spezifischen Plugins für jedes Betriebssystem auflisten, da wir in unseren Fall einen Speicher-Dump von Windows analysieren; listen wir die Windows-Plugins auf; indem wir Volatility mit dem Schlüsselwort `windows` als Argument ausführen, um Windows-Plugins mit den Schalter `--help` aufzulisten, wie unten gezeigt.

```
Volatility3
user@machine$ vol windows --help
usage: volatility [-h] [-c CONFIG] [--parallelism [{processes,threads,off}]] [-e EXTEND] [-p PLUGIN_DIRS] [-s SYMBOL_DIRS] [-v] [-l LOG] [-o OUTPUT_DIR] [-q] [-r RENDERER] [-f FILE] [--write-config] [--save-config SAVE_CONFIG] [--clear-cache] [--cache-path CACHE_PATH] [--offline]
                  [--single-location SINGLE_LOCATION] [--stackers [STACKERS [STACKERS ...]]] [--single-swap-locations [SINGLE_SWAP_LOCATIONS [SINGLE_SWAP_LOCATIONS ...]]]
                  plugin ...
volatility: error: argument plugin: plugin windows matches multiple plugins (windows.bigpools.BigPools, windows.cachedump.Cachedump, windows.callbacks.Callbacks, windows.cmdline.CmdLine, windows.crashinfo.Crashinfo, windows.devicetree.DeviceTree, windows.dlllist.DllList, windows.driverirp.DriverIrp, windows.drivermodule.DriverModule, windows.driverscan.DriverScan, windows.dumpfiles.DumpFiles, windows.envars.Envars, windows.filescan.FileScan, windows.getservicesids.GetServiceSIDs, windows.getsids.GetSIDs, windows.handles.Handles, windows.hashdump.Hashdump, windows.info.Info, windows.joblinks.JobLinks, windows.ldrmodules.LdrModules, windows.lsadump.Lsadump, windows.malfind.Malfind, windows.mbrscan.MBRScan, windows.memmap.Memmap, windows.mftscan.ADS, windows.mftscan.MFTScan, windows.modscan.ModScan, windows.modules.Modules, windows.mutantscan.MutantScan, windows.netscan.NetScan, windows.netstat.NetStat, windows.poolscanner.PoolScanner, windows.privileges.Privs, windows.pslist.PsList, windows.psscan.PsScan, windows.pstree.PsTree, windows.registry.certificates.Certificates, windows.registry.hivelist.HiveList, windows.registry.hivescan.HiveScan, windows.registry.printkey.PrintKey, windows.registry.userassist.UserAssist, windows.sessions.Sessions, windows.skeleton_key_check.Skeleton_Key_Check, windows.ssdt.SSDT, windows.statistics.Statistics, windows.strings.Strings, windows.svcscan.SvcScan, windows.symlinkscan.SymlinkScan, windows.vadinfo.VadInfo, windows.vadwalk.VadWalk, windows.vadyarascan.VadYaraScan, windows.verinfo.VerInfo, windows.virtmap.VirtMap)
```

Plugins sind während der Analyse mit Volatility3 äußerst hilfreich, da sie einen Speicher-Dump schnell nach bestimmten Datentypen durchsuchen und die Daten gemäß dem ausgewählten Plugins sortieren. Eine Zusammenfassung einiger der wichtigsten Plugins finden Sie unten:



| Plugin | Beschreibung |
|:---------|:---------|
| `windows.cmdline` | Listet die Kommandozeilenargumente von Prozessen auf |
| `windows.drivermodule` | Ermittelt, ob geladene Treiber von einem Rootkit versteckt wurde |
| `windows.filescan` | Scannt nach Dateiobjekten, die in einem bestimmten Window-Speicherabbild vorhanden sind |
| `windows.getsids` | Zeigt die SIDs (Security Identifiers) an, denen jeder Prozess gehört |
| `windows.handles` | Listet die offenen Handles von Prozessen auf |
| `windows.info` | Zeigt OS- & Kernel-Details des zu analysierenden Speicher-Samples an |
| `windows.netscan` | Scannt nach Netzwerkobjekten, die in einem bestimmten Windows-Speicherabbild vorhanden sind |
| `windows.netstat` | Durchläuft Netzwerkverfolgunsstrukturen, die in einem bestimmten Windows-Speicherbild vorhanden sind |
| `windows.mftscan` | Scannt nach Alternate Data Streams (ADS) |
| `windows.pslist` | Listet die Prozesse auf, die in einem bestimmten Windows-Speicherabbild vorhanden sind |
| `windows.pstree` | Listet-Prozesse in einer Baumstruktur basierend auf ihrer übergeordneten Prozess-ID auf |


Jetzt, da wir wissen, wie wir auf unsere Umgebung zugreifen und welche Tolls wir verwenden werden, gehen wir zur nächsten Phase über, in der wir mit der Analyse der Daten beginnen.



## **6. Informationen sammeln**

Informationen über das Ziel zu erhalten, ist entscheidend für unsere Untersuchung, da es sicherstellt, dass wir den richtigen Kontext und die richtige Umgebung der Beweismittel analysieren. Dieser Schritt hilft uns, spezifishce Architekturen und Betriebssystem zu verstehen, wodurch die Genauigkeit, Relevanz und Legitimität unsere Ergebnisse gewährleistet wird.

Wir können Informationen über das Zeil erhalten, indem wir den Schalter `-f` verwenden, um die analysierende Datei anzugeben, in diesem Fall `memdump.mem`, gefolgt vom Plugin `windows.info`, das verwendet wird, um die allgemeinen Informatione zu erhalten, wie im folgenden Beispiel gezeigt.

```
Volatility
user@machine$ vol -f memdump.mem windows.info
Volatility 3 Framework 2.5.2
Progress:  100.00PDB scanning finished                        
Variable                           Value

Kernel Base                        0xf9066161c000
DTB                                0x1ac000
Symbols                          file:///home/analyst/volatility3-2.5.2/volatility3/symbols/windows/ntkrnlmp.pdb/4DBE144182FF4156845CD3BD8B65
4E56-1.json.xz
Is64Bit                            False
IsPAE                              False
layer_name                         0 WindowsIntel32e
memory_layer                       1 FileLayer
KdVersionBlock                     0xf8066222a400
Major/Minor                        15.19041
MachineType                        34404
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

Die obige Ausgabe zeigt relevante Informationen zur Identifizierung der Maschine, an der wir arbeiten, z.B. *Architektur*, *Anzahl der Prozessoren*, und *Version*. Alls dies kann uns helfen, Informationen und Daten mit anderen Analysen zu korrelieren, die auf seperater Hardware der kompromittierten Maschine oder des Netzwerks selbst durchgeführt wurden, während wir immer noch den Nachweis haben, dass die kompromittierte Maschine war.

Unter Berücksichtigung dessen navigieren Sie zum **Arbeitsverzeichnis** in der VM und führen Sie den Befehl `vol -f memdump.mem windows.info` aus, um die nächsten Fragen zu beantworten.


## **7. Verdächtige Aktivitäten identifizieren**

Nachdem wir nun die Informationen über das Zielsystem haben, an dem wir arbeiten, versuchen wir, verdächtige Aktivitäten im Speicher-Dump zu identifizieren.

### **7.1 Verdächtige Aktivität**

Verdächtige Aktivitäten sind auf technische Anomalien, die in einem System vorhanden sein können, z.B. **unerwartete Prozesse, ungewöhnliche Netzwerkverbindungen oder Registierungsmodifikationen**. Diese Aktivitäten signalisieren oft potenzielle Sicherheitsbedrohungen wie Malware-Angriffe oder Datenlecks.

### **7.2 Die Suche beginnt**

Wir können damit beginnne, potenzielle Netzwerkaktivitäten zu beobachten. Wir können das Plugin `windows.netstat` verwenden, um zu sehen, ob es eine interessante oder ungewöhnliche Verbindung gibt. In diesem Stadium sind Fernzugirffsverbindungen oder Zugriffe auf verdächtige Websites etwas, wonach wir suchen sollten.

Navigieren wir zum Verzeichnis `/home/analyst` und führen wir den Befehl aus:

```
vol -f memdump.mem windows.netstat
```

Aus dem obigen Angaben können wir eine Verbindung beobachten, die auf Port `3389` von der IP `192.168.182.139` mit dem Zeitstempel `2024-02-24 22:47:52.00` hergestellt wure; dies könnte eine **anfänglichen Zugriff des Angreifers** anzeigen.

Nachdem wir nun Informationen über das Netzwerk haben, schauen wir uns die Prozesse an. Ein Volatilily-Plugin das wir verwenden können, ist `windows.pstree`, das ein Baum auf dem Betriebssystem laufenden Prozesse anzeigt.

```
vol -f memdump.mem windows.pstree

//Beispiel
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
0xffffa3028c24c580     **update_utility.exe** 3120   1200     45     320       1      False  2025-07-25 10:15:22.000000  <-- Verdächtig: child of explorer.exe
0xffffa3028c24c600       **winupdater.exe** 4560   3120     15     180       1      False  2025-07-25 10:15:40.000000  <-- Verdächtig: child of update_utility.exe
0xffffa3028c24c680         **nc.exe** 5000   4560      5      50        1      False  2025-07-25 10:15:55.000000  <-- Hochverdächtig: child of winupdater.exe, bekannter Netcat-Prozess
0xffffa3028c24c700           cmd.exe        5500   5000      2      30        1      False  2025-07-25 10:16:10.000000
0xffffa3028c24c780             powershell.exe 6000   5500     10     100       1      False  2025-07-25 10:16:30.000000
0xffffa3028c24c800   notepad.exe          2800   1200     10      80        1      False  2025-07-25 09:30:10.000000
0xffffa3028c24c880   msteams.exe          3500   1200    100     800       1      False  2025-07-25 09:10:00.000000
```

Wie wir der obigen Ausgabe entnehmen können, liefert uns der Befehl Informationen über hierarchisch auf dem System laufende Prozesse, die uns den Prozess und dessen jeweiligen übergeordneten Prozesse anzeigen. In diesem Fall ist `Services.exe` der übergeordnete Prozess von `dllhost.exe`.

Aber wie können wir einen verdächtigen Prozess identifizieren? Eine der häufigsten Methoden ist die Überprüfung des Namens des Prozesses; Bedrohungsakteure verwenden häufig Namen, um die Ausführung zu verschleiern. Eine Möglichkeit, dies zu tun, besteht darin, zu überprüfen, ob dieser Prozess normalerweise nicht vorhanden ist. Eine Lister der in Windows häufig verwendeten Prozesse finden Sie in diese Tabelle:

| Prozessname                | Beschreibung                                                                                                   |
|----------------------------|----------------------------------------------------------------------------------------------------------------|
| `System`                     | Ein Kernel-Level-Prozess, der Kernelfunktionen und die Kommunikation zwischen Hardware und Betriebssystem verwaltet. |
| `Systemunterbrechungen`      | Zeigt Hardware-Interrupts an, reagiert auf Geräteanfragen. Hohe Werte können auf Hardwareprobleme hinweisen.   |
| `svchost.exe`                | Generischer Hostprozess für Windows-Dienste, startet und verwaltet mehrere Systemdienste.                      |
| `explorer.exe `              | Windows-Explorer, verwaltet Desktop, Taskleiste, Startmenü und Dateioperationen.                               |
| `csrss.exe   `               | Client/Server Runtime Subsystem, unterstützt Konsolen- und GUI-Anwendungen, verwaltet Threads und Fenster.     |
| `winlogon.exe `              | Verarbeitet Benutzeranmeldung/-abmeldung, steuert „Ctrl+Alt+Delete“ und den Sperrbildschirm.                   |
| `taskhostw.exe`              | Hostprozess für DLL-basierte Dienste, startet Windows-Dienste, die von DLLs abhängen.                          |
| `services.exe`               | Service Control Manager, verwaltet Ausführung und Status von Windows-Diensten.                                 |
| `lsass.exe     `             | Local Security Authority Subsystem Service, verwaltet Sicherheitsrichtlinien, Authentifizierung und Passwörter.|
| `smss.exe`                   | Session Manager Subsystem, erster Prozess nach Boot, startet Kerneldienste und Systemtreiber.                  |
| `spoolsv.exe`                | Druckwarteschlangenprozess, verwaltet und speichert Druckaufträge.                                             |
| `dwm.exe                    `| Desktop Window Manager, verwaltet grafische Effekte und Fensteranimationen.                                    |
| `RuntimeBroker.exe          `| Verwaltet Berechtigungen von Windows-Apps, schützt Privatsphäre und Ressourcen.                                |
| `audiodg.exe                `| Isoliert Audioverarbeitung von anderen Systemprozessen, schützt Audio vor Systemabstürzen.                     |
| `sihost.exe                 `| Shell Infrastructure Host, verwaltet grafische Elemente wie Startmenü und Taskleiste.                          |
| `wininit.exe                `| Startet Systemdienste und mountet Laufwerke beim Hochfahren.                                                   |
| `SearchIndexer.exe          `| Windows Search Indexer, erstellt Index für schnellere Suchvorgänge.                                            |
| `WUDFHost.exe               `| User-Mode Driver Framework Host, verarbeitet benutzerdefinierte Gerätetreiber.                                 |
| `TrustedInstaller.exe       `| Windows Module Installer, verwaltet Installation und Updates von Windows-Komponenten.                           |
| `mdm.exe                    `| Mobile Device Management, verwaltet mobile Geräte und deren Synchronisation.                                   |
| `conhost.exe                `| Console Window Host, unterstützt Konsolenumgebung und Befehlszeilenprogramme.                                  |
| `fontdrvhost.exe            `| Usermode Font Driver Host, verwaltet Schriften und deren Darstellung.                                          |
| `msmpeng.exe                `| Windows Defender Antivirus Service, Echtzeitüberwachung und Schutz vor Malware.                                |
| `mstsc.exe                  `| Remote Desktop Client, ermöglicht Fernsteuerung über das Netzwerk.                                             |
| `rundll32.exe               `| Führt Funktionen aus DLLs aus, wichtig für Funktionsaufrufe in Windows.                                        |
| `ctfmon.exe                 `| CTF Loader, verwaltet alternative Eingabemethoden wie Sprache und Bildschirmtastatur.                          |
| `CompatTelRunner.exe        `| Compatibility Telemetry Runner, sammelt Telemetriedaten für Microsoft.                                         |
| `sedsvc.exe                 `| Windows Remediation Service, behebt Probleme mit Windows-Updates.                                              |
| `musnotification.exe        `| Modern Update Service, verwaltet Update-Benachrichtigungen.                                                    |
| `SecHealthUI.exe            `| Stellt die Benutzeroberfläche der Windows-Sicherheitsanwendung bereit.                                         |
| `LogonUI.exe                `| Verwaltet Sperr- und Anmeldebildschirm, ermöglicht Anmeldung mit Passwort, PIN oder Biometrie.                 |
| `BackgroundTaskHost.exe     `| Verwaltet Hintergrundaufgaben für Apps und systemnahe Anwendungen.                                             |
| `nvcontainer.exe            `| NVIDIA-Prozess zur Verwaltung von Treiber- und Grafikeinstellungen.                                            |
| `appvclient.exe             `| Microsoft App-V Client, verwaltet virtuelle Anwendungen in Windows.                                            |
| `GameBarPresenceWriter.exe  `| Teil der Xbox Game Bar, verwaltet Spielaufzeichnungen und Übertragungen.                                       |
| `dasHost.exe                `| Device Association Framework Provider Host, verwaltet Kommunikation mit angeschlossenen Geräten.                |
| `wmiprvse.exe               `| WMI Provider Host, stellt systemweite Informationen für Anwendungen bereit.                                    |
| `sppsvc.exe                 `| Software Protection Service, verwaltet Lizenzprüfung und -aktivierung.                                         |
| `lsm.exe                    `| Local Session Manager, verwaltet Benutzeranmeldungen und Sitzungen.                                            |
| `mmc.exe                    `| Microsoft Management Console, zentrale Oberfläche für Systemadministratoren.                                   |
| `inetinfo.exe               `| IIS Admin Process, verwaltet Webserver-Dienste und -Anwendungen.                                               |
| `wbemcomn.exe               `| WMI-Prozess, ermöglicht Verwaltung und Zugriff auf Systeminformationen.                                        |
| `unsecapp.exe               `| Verarbeitet WMI-Anfragen für Kommunikation zwischen Anwendungen und Remote-Systemen.                           |
| `taskeng.exe                `| Task Scheduler Engine, verwaltet geplante Aufgaben und Systemwartungen.                                        |
| `pcalua.exe                 `| Program Compatibility Assistant, überwacht Anwendungen auf Kompatibilitätsprobleme.                            |
| `wlanext.exe                `| Verwaltet WLAN-Verbindungen und deren Konfiguration.                                                           |
| `dllhost.exe                `| COM Surrogate, hostet COM-Objekte für Windows und Anwendungen.                                                 |
| `msiexec.exe                `| Windows Installer, verwaltet Installation, Reparatur und Deinstallation von Programmen.                        |
| `rdrservices.exe            `| Remote Desktop Services, verwaltet Remotedesktopverbindungen.                                                  |

In Anbetracht des oben Gesagten und der erneuten Betrachtung der Ausgabe können wir einen Prozess mit dem abgeschnittenen Namen `critical_updat` beobachten.

```
PID    PPID   Name             Address         Threads  Handles  Session  Wow64
3384   7960   conhost.exe      0xe50edab37080  4        -        1       False
1648   7960   critical_updat   0xe50ed9dc1080  5        -        1       False
1648   7960   updater.exe      0xe50edab53080  6        -        1       False
6460   3196   FTK Imager.exe   0xe50edad09080 19        -        1       False
```

Dieser Prozess sieht nicht so aus, als ob er Teil des Systems wäre, und bei genauerer Betrachtung ist der übergeordnete Prozess von `update.exe`, der ebenfalls nicht als Prozess, der Teil des Windows-Betriebssystem ist, aufgeführt ist.

Großartig. wir identifizieren einen möglichen bösartigen Prozess und sollten die Informatione wie **Zeitstempel, PID, PPID** und **Memory-Offset** notieren.


## **8. Detaillierte Untersuchung des Prozesses `update.exe`**

Mit den gesammelten Informationen können wir den Prozess `critical_updat` untersuchen, den wir in unseren vorherigen Aufgaben identifiziert haben un der einen Kindprozess namens `updater` hat. Lassen Sie uns den Kindprozess genauer untersuchen. Beginnen wir damit, zu schauen, wo auf der Festplatte er gespeichert wurde; dafür können wir das Plugin `windows.filescan` verwenden, das uns erlaubt, die auf dem Speicher-Dump zugreifenden Datei zu untersuchen. Diese Ausgabe ist ziemlich groß, daher werden wir den `>`-Charakter in Bash verwenden, um die Ausgabe zu besseren Anzeige in eine Datei umzuleiten, in diesen Fall `filescan_out`.

```
vol -f memdump.mem windows.filescan > filescan_out
```

Nachdem der Befehl ausgeführt wurde, können wir die Datei mit `cat` inspizieren und mit dem `grep`-Befehl filtern, wie unten gezeigt.

```
cat filescan_out |grep updater
```

Aus der obigen Ausgabe können wir beobachten, dass die Datei im Verzeichnis `\Users\user01\Documents\updater.exe` oder `C:\Users\user01\Documents\updater.exe` gespeichert wurden.

Wenn wir detailliertere Informationen wie den Zugriffs- oder Änderungszeitpunkt der Datei wünschen, können wir das Plugin `windows.mftscan.MFTScan` verwenden, dessen Ausgabe ebenfalls recht groß ist, sodass wir die Ausgabe in die Datei `mftscan_out` umleiten werden, wie unten gezeigt.

```
vol -f memdump.mem windows.mftscan.MFTScan > mftscan_out
```

Wir können dann den `grep`-Befehl erneut verwenden, um die Datei nach dem Auftreten von `updater.exe` zu durchsuchen.

```
cat msfscan_out | grep updater
```

Aus der obigen Ausgabe beobachten wir, dass die letzten vier Zeitstempel den **Erstellungs-, Änderungs-, Update- und Zugriffszeitstempeln** entsprechen; wir können und dies notieren.

### **8.1 Die "Beute" erhalten**

LassenSie uns Informationen über den Prozess erhalten. Es gibt meherere Möglichkeiten, den Speicher zu untersuchen, aber wir werden weiterhin Volatility verwenden. Diesmal werden wir den Speicherbereich, der zu `updater.exe` gehört, dumpen und untersuchen.

Um dies zu erreichen, verwenden wir das Plugin `windows.memmap`. Diesmal werden wir das Ausgabeverzeichnis mit dem `-o`-Schalter angeben. In diesem Fall verwenden wir dasselbe Verzeichnis, das durch das Zeichen `.` bezeichnet wird, um die Option `--dump` gefolgt von der Option `--pid` und der PID des Prozesses, die im Fall von `updater.exe` **1612** ist.

```
vol -f memdump-mem -o . windows.memmap --dump --pid 1612
```

Wenn der obige Befehl beendet ist, werden wir eine Datei mit der Erweiterung `.dmp` in unseren Arbeitsverzeichnis haben.

Die Untersuchung der Datei ist schwierig, da sie nicht druckbare Zeichen enthält, daher verwenden wir den `string`-Befehl, um die Ausgabe besser zu analysieren. Da uns die Dateistrings jetzt zur Verfügung stehen, können wir nach Schlüsselwortmustern wie `HTTP` oder `key` oder einem beliebigen Muster suchen, das uns schnell zu ein Artefakt führen kann. Eine andere Möglichkeit, durch das Terminal zu scrollen, in diie Verwendung des `strings`-Befehls, der an `less` weitergeleitet wird, um durch die Ausgabe zu navigieren, wie in der Befehlsausgabe unten gezeigt.

```
strings pip.1612.dmp | less                                         // Kann sehr mühsellig sein, alles genau zu durchsuchen
strings pip.1612.dmp | grep "HTTP"                                  // Zeigt alle HTML Elemente in der .dmp Datei an
```

Wie wir beobachten können, haben wir sofort einen möglichebn Schlüssel und eine Domäne von einer URL identifiziert, auf die der Prozess möglicherweise zugegriffen hat. Auch durch Herunterscrollen / `grep '.pdf'` fanden wir weitere Hinweise darauf, dass dies ein bösartiger Prozess ist, da wir den Dateinamen `important_document.pdf` finden, der eine Interaktion mit der Datei zeigt.

Großartig, wir können folgern, dass der Prozess `updater.exe` auf das Dokument `important_document.pdf` zugegriffen und zu irgendeinem Zeitpunktz auf einen "Schlüssel" unter der URL `http://key.critical-update.com/encKEY.txt` zugegriffen hat.

Wenn wir den Befehl `grep` verwenden, um nach der HTTP-Anfrage zu suchen, die möglicherweise im Speicher gespeichert ist, können wir dies mit `-B` und `-A` tun, um 10 Zeilen oberhalt und unterhalb unserer Übereinstimmung zu suchen, um zu sehen, ob wir noch etwas entdecken können.

```
strings pid.1612.dmp | grep -B 10 -A 10 "http://key.critical-update.com/encKEY.txt"
```

Beim Hochscrollen können wir die HTTP-Anfrage beobachten, wie angezeigt.

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

Aus dem oben Gennanten können wir am Ende der HTTP-Anfrage den Inhalt der Datei `encKey.txt` beobachten, und in derselben Anfrage können wir Daten mit Wert `cafebabe` beobachten. Dies könnte der **Schlüssel** seubm der vin Abgreufer zur Verschlüsseln der PDFs verwendet wurde und nicht auf die Festplatte heruntergeladen wurde.

Ausgezeichnet. Wir haben wertvolle Informationen aus dem Speicher-Dump gesammelt, einschließlich des möglichen Schlüssels, der zum Verschlüsseln der Dokumente verwendet wurde.


----

<div align="center">

© 2025 by daonware 
<br>Erstell: July 25, 2025
<br>Zuletzt geupdatet: July 25, 2025

License: [Public Domain / CC0](https://creativecommons.org/publicdomain/zero/1.0/)

</div>