
---

# Initial Setup

An overview of the Windows features and software we will use throughout this course. Learn how to configure your own system so that you can follow along.

---

💻 **Attention Linux and Mac Users:**

If you are using Linux, you can run a Windows 11 virtual machine using VMware Workstation Pro (free) or any equivalent virtualization software.

If you are using macOS, you can run a Windows 11 virtual machine using either VMware Fusion (free) or Parallels Desktop (paid, with a free 14-day trial). On Apple Silicon-based Macs, you will need to use Windows 11 on Arm. Forensic artifacts behave the same in Windows 11 on Arm as those in the x64 version, so you will be able to follow along with the course without issue.

See the "VMware Downloads" and "Parallels Downloads" sections below. Note that in either case, you can use an unlicensed Windows 11 installation for learning purposes.

---

**⚙️ Windows Features:**

**Windows Sandbox (WSB)**

Windows Sandbox provides a temporary, disposable Windows environment. Each time you launch it, a clean virtual machine starts. You can install or remove software, change settings, and test forensic artifacts safely. Everything resets when you close it.

Note: Windows Sandbox requires Windows 11 Pro or Enterprise and is not available on Home editions. If you do not have access to Sandbox, VMware Workstation Pro (free) is a full alternative for running isolated test environments. See the VMware Downloads section below.

**If you are running Windows inside a virtual machine on a Linux or Mac host, you do not need to enable Windows Sandbox.**

1. Press `Win + R`, type `optionalfeatures`, and press Enter.  
2. In Windows Features, check Windows Sandbox.  
3. Click OK and restart when prompted.  
  

**Windows Subsystem for Linux (WSL)**

Windows Subsystem for Linux (WSL) allows you to run a full Linux distribution directly within Windows, without needing a separate virtual machine. It provides access to familiar Linux tools and command line utilities, making it easier to analyze artifacts, parse logs, and perform scripting tasks alongside native Windows utilities.

1. Open PowerShell as Administrator.  
2. Type `wsl --install`  
3. Restart when prompted. Ubuntu 24.04 LTS will be installed by default.

Once you have enabled the required Windows features, install the following tools before continuing with the course.

---

**🛠️ Tools:**

- [Eric Zimmerman's Tools](https://ericzimmerman.github.io/#!index.md)
    
    - A collection of powerful Windows forensic utilities for parsing event logs, registry hives, prefetch files, and many other artifacts. Use Get-ZimmermanTools to download all programs at once and keep your tool set current.
        
- [KAPE](https://www.kroll.com/en/services/cyber-risk/incident-response-litigation-support/kroll-artifact-parser-extractor-kape)
    
    - A triage and evidence collection tool that quickly gathers and processes targeted data from live or mounted systems.
        
- [Arsenal Image Mounter](https://arsenalrecon.com/downloads)
    
    - Mounts forensic disk images in Windows, allowing you to interact with evidence as if it were a local disk.
        
- [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)
    
    - A trusted tool for acquiring forensic images, viewing disk contents, and exporting individual files or folders from evidence.
        
- [RegRipper 3.0](https://github.com/keydet89/RegRipper3.0) | [RegRipper 4.0](https://github.com/keydet89/RegRipper4.0)
    
    - Extracts and interprets data from Windows registry hives using plugin-based parsing for key forensic artifacts. You may use either version for this course, but RegRipper 3.0 is more widely adopted.
        
- [Sysinternals Suite](https://www.microsoft.com/store/productId/9P7KNL5RWT25?ocid=pdpshare)
    
    - A comprehensive suite of system utilities from Microsoft for monitoring, troubleshooting, and analyzing Windows internals.
        
- [PhotoRec](https://www.cgsecurity.org/wiki/TestDisk_Download)
    
    - An open-source tool for carving deleted files from a variety of file systems.
        
- [NirSoft BrowsingHistoryView](https://www.nirsoft.net/utils/browsing_history_view.html)
    
    - Aggregates browsing history from multiple browsers and profiles into a single, easy-to-read view.
        
- [Thumbs Viewer](https://thumbsviewer.github.io/)
    
    - Parses legacy `Thumbs.db` files used by older versions of Windows to store thumbnail images.
        
- [Thumbcache Viewer](https://thumbcacheviewer.github.io/)
    
    - Displays and exports thumbnail images stored in Windows thumbnail cache databases.
        

**➕ Additional tools will be installed as needed in later lessons.**

---

**📦 VMware Downloads:**

**Broadcom has made VMware Workstation Pro (Windows/Linux) and Fusion (macOS) available as free products. To use either one, follow the links below to create a Broadcom account and download the software.**

- [Broadcom Support Portal: User Registration (Required to Download)](https://profile.broadcom.com/web/registration)
    
- [VMware Workstation Pro](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Workstation%20Pro&freeDownloads=true)
    
- [VMware Fusion](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20Fusion&freeDownloads=true)
    

---

**📦 Parallels Downloads:**

**Parallels Desktop is a macOS virtualization solution recommended for Apple Silicon users. Although it is a paid product, a 14-day free trial is available via the link below.**

- [Parallels Desktop](https://www.parallels.com/products/desktop/)

---

# Event Logs In-depth Analysis

A very in-depth walkthrough of how to view, filter, and analyze Windows Event Logs. We'll look at numerous scenarios and use a cheat sheet to reference many of the most important Channels, Providers, and Event IDs.

**Resources:**  
[Windows Event Log Cheat Sheet](https://cdn.13cubed.com/downloads/windows_event_log_cheat_sheet.pdf)  
[RDP Flowchart](https://cdn.13cubed.com/downloads/rdp_flowchart.pdf)

## How to install and setup sysmon

Step-by-Step Implementation:

1. **Download Sysmon**: Get `sysmon64.exe` (for 64-bit) from Microsoft's Sysinternals page.
2. **Get a Configuration File**: Download a pre-built configuration (e.g., SwiftOnSecurity's, [Olaf](https://www.google.com/search?q=Olaf&oq=implementing+sysmon+on+windows&gs_lcrp=EgZjaHJvbWUyBggAEEUYOdIBCTk0MTFqMGoxNagCCLACAQ&sourceid=chrome&ie=UTF-8&mstk=AUtExfBlAeP9Hftw-zoJaGm3OazENwr7YMNlIYQ6bxiUQZmbm6WhaURXaEN7KrF-Yp0rXmDC_Qpo8Y_ZGeQi2g74ivWNjH9nHkf-gGylNsomhdFtGTrjMjRrAyIUhRHEWs3MjMk&csui=3&ved=2ahUKEwi7n_nklKyRAxW248kDHSYOAIAQgK4QegQIAxAC)'s) from GitHub to define what to log.
3. **Open PowerShell as Administrator**: Right-click PowerShell and select "Run as administrator".
4. **Navigate to Sysmon Folder**: Use `cd C:\Path\To\Sysmon`.
5. **Install Sysmon**: Run `sysmon64.exe -i sysmonconfig.xml` (replace `sysmonconfig.xml` with your config file's name) and agree to the EULA.
6. **Verify Installation**:
    - Open Event Viewer (search for it in the Start Menu).
    - Navigate to `Applications and Services Logs` > `Microsoft` > `Windows` > `Sysmon` > `Operational`.
    - Look for events (like Event ID 1 for process creation) to confirm it's working.

Sysmon manual install video:
https://www.youtube.com/watch?v=98B9UmFr0qs&t=187s

---

# Event Logs Tools and Best Practices

Learn how to use Eric Zimmerman's EvtxECmd to parse Windows Event Logs and analyze the output with Timeline Explorer. Then, we'll talk about logging in a perfect world, and what an ideal baseline configuration might look like.  
  
**Resources:**  
[Windows Baseline Logging](https://nullsec.us/windows-baseline-logging/)

---

# Registry NTUSER.DAT

An in-depth look at the NTUSER.DAT registry hive, its purpose, and a walkthrough of some of the most forensically important keys and values therein. This lesson also introduces Eric Zimmerman's Registry Explorer.  
  
**Resources:**  
[Windows Registry Cheat Sheet](https://cdn.13cubed.com/downloads/windows_registry_cheat_sheet.pdf)

---

# Registry Usrclass.dat and ShellBags

In ShellBags Explorer, Unmapped GUID means the tool found a ShellBags entry it could not translate into a normal folder path. This often happens with virtual folders (like Control Panel or Recycle Bin), deleted or moved folders, or objects identified only by GUIDs. It is not an error, just ShellBags Explorer showing you that the entry cannot be mapped to a standard location.  
  
This write-up explains in more detail if you are interested:  
[https://www.4n6k.com/2018/07/forensics-quickie-detailing-shell-item-guid-behavior-creative-cloud.html](https://www.4n6k.com/2018/07/forensics-quickie-detailing-shell-item-guid-behavior-creative-cloud.html)

---

# Evidence of Execution - Shimcache-AppCompatCache

An in-depth look at one of the most confusing Windows artifacts. There is a lot you need to know about Shimcache/AppCompatCache. Learn how to avoid the pitfalls and master the use of this important source of information.

**In August 2024, this lesson was updated with important information and new research from** [**nullsec.us**](http://nullsec.us/) **and 13Cubed. While this artifact remains most valuable for showing the presence or existence of a file, it can, in some cases, be used to show execution—even in Windows 10 and later!**

**Also note that Volatility 3 now includes a plugin called** `windows.shimcachemem` **that parses this artifact from memory.**

**For even more in-depth information, please refer to this multipart blog series:**  
[AppCompatCache - Part 1](https://nullsec.us/windows-10-11-appcompatcache-deep-dive/)  
[AppCompatCache - Part 2](https://nullsec.us/building-appcompatcacheparser/)  
[AppCompatCache - Part 3](https://nullsec.us/appcompatcache-part-3/)

---

# Evidence of Execution - Amcache

Learn about the importance of AmCache and how it is unique among all of the other artifacts in this module with regards to the kinds of information it can provide us.

**Resources:**  
[Analysis of the AmCache](https://cyber.gouv.fr/sites/default/files/2019/01/anssi-coriin_2019-analysis_amcache.pdf)

---

# Evidence of Execution - PCA

Learn about the Program Compatibility Assistant (PCA), one of the newest evidence of execution artifacts introduced in Windows 11 22H2.

**Resources:**  
[New Windows 11 Pro (22H2) Evidence of Execution Artifact!](https://aboutdfir.com/new-windows-11-pro-22h2-evidence-of-execution-artifact/)

In **November 2023**, Sygnia published new information about this artifact that expands on the initial research:  
[Diving Into the New Windows 11 PCA Artifact](https://www.sygnia.co/blog/new-windows-11-pca-artifact/)

---

# Persistence, Privilege Escalation, and Lateral Movement - SMB, RDP, WMI, PsExec, UAL


Learn about the numerous methods by which a threat actor could perform lateral movement within a victim environment, including: SMB, RDP, WMI, PsExec, and Impacket; an introduction to User Access Logging (UAL) - a critical Windows feature/artifact that can be leveraged to detect lateral movement.  
  
**Resources:**  
[Impacket Exec Commands Cheat Sheet](https://cdn.13cubed.com/downloads/impacket_exec_commands_cheat_sheet.pdf)  
[Impacket Exec Commands Cheat Sheet (Poster)](https://cdn.13cubed.com/downloads/impacket_exec_commands_cheat_sheet_poster.pdf)

---

# Anatomy of NTFS - Metafiles, MFT, Journaling, ADS

Learn about the building blocks of the NTFS file system; the MFT and MFT FILE records; file system journaling, and Alternate Data Streams (ADS).  
  
**Resources:**  
[Anatomy of an NTFS FILE Record](https://cdn.13cubed.com/downloads/anatomy_of_an_ntfs_file_record.pdf)

---

# Anatomy of NTFS - MACB Timestamps

An in-depth look at $STANDARD_INFORMATION and $FILE_NAME Modification, Access, MFT Record Change, and Birth (MACB) timestamps and how they behave based upon a variety of operations; important timestamp changes in Windows 10/11.  
  
**Resources:**  
[SANS Windows Forensic Analysis Poster](https://www.sans.org/posters/windows-forensic-analysis/)  
[Windows 11 Time Rules](https://www.khyrenz.com/post/windows-11-time-rules)

---

# File Deletion and Recovery - File Carving with PhotoRec

Learn how to perform file carving using PhotoRec, and explore how to efficiently and quickly search through recovered data using the Windows Subsystem for Linux (WSL).

Note that the "image.raw" disk image used in the demo portion of this lesson is also the one used for the Knowledge Assessment. You may download it [here](https://files.13cubed.com/iwe/image.zip) if you want to follow along.

**Resources:**  
[TestDisk/PhotoRec Documentation](https://www.cgsecurity.org/testdisk_doc/)  
[Creating a Custom Signature for PhotoRec](https://www.cgsecurity.org/testdisk_doc/photorec_custom_signature.html)

In addition to the methods used in this lesson, I reckon using YARA rules can also get you closer to the result(s) you want, dependent on its strength and how specific or loose you written it. Ideally in forensic investigations you would want them as loose as possible. You can either write up one yourself or ask your threat intelligence teams in case you need to attribute the case to a particular adversary or tooling.  
  
YARA:  
- [https://github.com/VirusTotal/yara](https://github.com/VirusTotal/yara)  
- [https://yara.readthedocs.io/en/latest/commandline.html](https://yara.readthedocs.io/en/latest/commandline.html)  
  
Example Command: `yara <rules_path> <target_path>`  
  
YARA-X (newer version, rust-based YARA)  
- [https://github.com/VirusTotal/yara-x](https://github.com/VirusTotal/yara-x)  
- [https://virustotal.github.io/yara-x/docs/cli/commands/](https://virustotal.github.io/yara-x/docs/cli/commands/)  
  
Example Command: `yr scan <rules_path> <target_path>`

---

# Timelining - Plaso/Log2Timeline

Learn how to harness the power of Plaso/Log2Timeline in digital forensics. This lesson provides a detailed guide to installing and using Plaso/Log2Timeline within the Windows Subsystem for Linux (WSL). You'll learn how to create comprehensive "super timelines," an essential resource for analyzing and correlating digital evidence.

**Resources:**  
[Plaso Documentation](https://plaso.readthedocs.io/en/latest/)  
[Digital Forensics Artifact Repository](https://github.com/ForensicArtifacts/artifacts)  
[Super Timeline for ACME-WIN11-WS-03](https://files.13cubed.com/iwe/timeline.zip)

---

# Additional Content - Web Browser Forensics

Learn the locations of common browser forensic artifacts within Microsoft Edge, Google Chrome, Mozilla Firefox, and more; learn how to parse history from numerous browsers with NirSoft's BrowsingHistoryView.  
  
**Resources:**  
[Windows Browser Artifacts Cheat Sheet](https://cdn.13cubed.com/downloads/windows_browser_artifacts_cheat_sheet.pdf)

---

# Additional Content - Thumbs.db and Thumbcache

Learn about Windows thumbnail cache databases (Thumbs.db and Thumbcache_*.db files), the potentially valuable data they may contain, and how to parse them.  
  
**Resources:**  
[Thumbs Viewer](https://thumbsviewer.github.io/)  
[Thumbcache Viewer](https://thumbcacheviewer.github.io/)

---

# Additional Content - Windows Activity Timeline

Learn about the Windows 10 Timeline, also known as the Windows Activity Timeline. Despite widespread misinformation about this artifact, the underlying `ActivitiesCache.db` database is present on some versions of Windows 11 and on Windows Server 2016/2019/2022. In some cases, it can serve as a valuable source of additional forensic metadata.

**Special thanks to Kobi Key for research used in this lesson.**

**Resources:**  
[How to Perform Clipboard Forensics: ActivitiesCache.db, Memory Forensics and Clipboard History](https://www.inversecos.com/2022/05/how-to-perform-clipboard-forensics.html)


---

# Additional Content - Windows Search Index

Learn about the Windows Search Index, a powerful artifact that can provide us with file metadata (paths, owners, timestamps, and sizes), Microsoft Edge history, program-specific user activity, per-user file opening, and even partial file contents.

**Resources:**  
[Windows Search Index: The forensic artifact you’ve been searching for](https://www.aon.com/cyber-solutions/aon_cyber_labs/windows-search-index-the-forensic-artifact-youve-been-searching-for/)  
[Search Index DB Reporter (SIDR)](https://github.com/strozfriedberg/sidr/releases/latest)


---
# Trouble at ACME

![Trouble at ACME](https://s3.us-west-2.amazonaws.com/content.podia.com/pz5arbur36ra8iw98zwk9u6njcrk)

**Trouble at ACME**

This challenge is comprised of two (2) disk and two (2) memory images, and as an enrolled student, you now have exclusive access to both the challenge documentation (PDF) and the accompanying images.

You'll be tasked with investigating a security alert that took place at ACME Corp. The PDF provides all the initial details required to kickstart your investigation. This challenge serves as an opportunity for self-assessment, allowing you to gauge your forensic abilities. Once you've reached the "**Stop Here!**" page, you can begin your investigation.

After completing your analysis and constructing a timeline of your findings, you'll have the chance to compare your results with the remainder of the document to evaluate your performance. It's important to note that, like real-world scenarios, uncovering 100% of the story may not always be feasible.

**To get started, see the "Trouble at ACME" PDF file at the link below.**

**📦 Downloads:**

- [trouble_at_acme.pdf](https://files.13cubed.com/iwe/acme/trouble_at_acme.pdf)
    
- [acme_disk1.zip](https://files.13cubed.com/iwe/acme/acme_disk1.zip)
    
- [acme_disk2.zip](https://files.13cubed.com/iwe/acme/acme_disk2.zip)
    
- [acme_memory1.zip](https://files.13cubed.com/iwe/acme/acme_memory1.zip)
    
- [acme_memory2.zip](https://files.13cubed.com/iwe/acme/acme_memory2.zip)

---

# Continue Your Learning 🎓

![The DFIR Report Logo](https://s3.us-west-2.amazonaws.com/content.podia.com/kk99nk0ztlus772l7ebbaccn265p)

**13Cubed has partnered with DFIR Labs from The DFIR Report.**

Level up your skills with hands-on investigations based on **real-world intrusions**. DFIR Labs gives you access to parsed SIEM data with real enterprise noise, multiple hosts, and quizzes that follow the intrusion kill chain. You’ll work with Windows Event Logs, Sysmon, detection alerts, memory artifacts, and more, all inside Splunk or Elasticsearch.

**Use code** **==13CUBED20==** **for 20% off your one-time access:** [**https://dfirlabs.thedfirreport.com/store**](https://dfirlabs.thedfirreport.com/store)

---

Investigating Windows Endpoints

**Earn your 13Cubed Certification!**

- There are **80** questions with a **total of 100 points** possible.  
    
- **60** multiple choice questions are based on course content, with a value of **1 point per question**.
- **20** multiple choice questions are based on disk image analysis, with a value of **2 points per question**.
    - You will need to **[download the disk image](https://files.13cubed.com/iwe/image.zip)**, mount it, and analyze it to answer the 20 practical questions. There is usually more than one way to obtain each answer, and you are free to use any of the methods and tools described in this course to answer them.
- This is a non-proctored exam delivered via Google Forms integration.
- There is no pre-defined time limit.
- If you fail the assessment three (3) times, you must wait a period of seven (7) days before you attempt again.
- A **minimum score of 70 points** is required to earn your certificate and digital badge. If you meet this requirement, you will receive your credentials via email from Certifier within three (3) business days.
    - A **gold badge** will be issued if you pass on the **first attempt**.
    - A **silver badge** will be issued if you pass on the **second attempt**.
    - A **bronze badge** will be issued if you pass on the **third (or later) attempt**.
    - Here is a [sample credential](https://credsverse.com/credentials/5d8c6f8a-0377-4002-a574-0a872ee60d0d).

**⚠️ IMPORTANT: Use the email address associated with your 13Cubed account.**

---

What is the difference between a Windows Event Log Channel and a Provider?  *  1 point


-A Channel is equivalent to the Windows Event Log filename and a Provider is a subcategory of log within the Channel

-A Provider is equivalent to the Windows Event Log filename and a Channel is a subcategory of log within the Channel

-Windows Event Logs use Channel names and Event IDs to track Windows events; a “Provider” is not associated with an Event Log

-Windows Event Logs use Provider names and Event IDs to track Windows events; a “Channel” is not associated with an Event Log

---

What Windows Event ID within the Service Control Manager Provider is associated with Service Installation?  *  1 point

-7035

-7045

-4697

-4624

-4688

---

What Windows Event Log Channel could be leveraged to determine possible NTDS.dit staging via ntdsutil?  *  1 point


-ESENT

-That information would not be found within a Windows Event Log

-NTDS

-Application

-ADDS

---

What Event ID could be leveraged within the System channel to find evidence of Event Log clearing (potential anti-forensics) for System and potentially other logs as well?  *  1 point


-1102

-Event ID 104 will show evidence of the System channel (log) being cleared, but NOT other logs

-Event ID 1102 will show evidence of the System channel (log) being cleared, but NOT other logs

-104

---

What does a 4624 Type 3 logon indicate when associated with an RDP connection?  *  1 point


-An RDP connection in which Network Level Authentication (NLA) is disabled

-A tunneled RDP connection

-An RDP connection using a port other than TCP/3389

-An RDP connection in which Network Level Authentication (NLA) is enabled

-Only Type 10 logons are associated with RDP connections

---

What three (3) Event IDs “chained together” could be used to detect a potential Pass-the-Hash attack on the _destination_ system?  *  1 point


-Windows Event Logs alone cannot be used to detect Pass-the-Hash

-4625, 4776

-4624 Type 5, 4672, 4776

-4624 Type 3, 4672, 4776

---

What are the user-specific Registry hives?

*

1 point

Amcache.hve and SOFTWARE

SOFTWARE and HKCU

NTUSER.DAT and UsrClass.dat

HKCU and HKLM

Which Registry hive contains the “Evidence of Execution” artifact MUICache?

*

1 point

UsrClass.dat

Amcache.hve

NTUSER.dat

SOFTWARE

What information can be gleaned from ShellBags?

*

1 point

A list of folders to which a given user has navigated within Windows Explorer

A list of folders from which programs have been executed

A full listing of files within a folder

A list of folders/directories navigated to from the Command Prompt

What files are tracked within ShellBags?

*

1 point

ZIP files, but not the contents within them, are tracked by ShellBags

ZIP files and the contents within them are tracked by ShellBags

Files are tracked by ShellBags, but only for locations visited via UNC/network path

No files are tracked by ShellBags – only folders visited via Windows Explorer

What Registry-based forensic artifact can be leveraged to determine files opened or saved via the Windows “common dialog” box?

*

1 point

RecentDocs

OpenSavePidlMRU

LastVisitedPidlMRU

TypedPaths

What Registry components have a “Last Write Timestamp” associated with them?

*

1 point

Values only

Values and Subvalues

Keys and Values

Keys only

What per-user Registry key tracks searches performed within the Windows Explorer “Search This PC” box (top-right of an Explorer window)?

*

1 point

Windows Search Database

RunMRU

RecentSearches

Search.edb

WordWheelQuery

What would the creation (B) timestamp for a given user’s NTUSER.DAT indicate?

*

1 point

No information with regards to a user logon could be determined via that timestamp alone

The last time the user logged in via an RDP session

The first time the user logged on to the system via any method

The first time the user interactively logged on to the system

The last time the user logged on to a system

What Registry key should be consulted to find the USB Vendor ID and Product ID of a USB flash drive that was connected to the system?

*

1 point

HKLM\SOFTWARE\Microsoft\Windows Portable Devices\Devices

HKLM\SYSTEM\MountedDevices

HKLM\SYSTEM\CurrentControlSet\Enum\USB

HKLM\SYSTEM\CurrentControlSet\Enum\USBSTOR

What Registry key should be consulted to find out if a particular user mounted a USB flash drive of interest?

*

1 point

HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\Mountpoints2

HKLM\SOFTWARE\Microsoft\Windows Portable Devices\Devices

HKCU\SOFTWARE\Microsoft\Windows NT\CurrentVersion\EMDMgmt

HKLM\SYSTEM\MountedDevices

What are Registry transaction logs?

*

1 point

They temporarily store keys, subkeys, and values that have been deleted from a given Registry hive

They serve as an intermediary prior to data being written to the hive files

They store metadata about the changes to a given Registry hive

They are used to store temporary information that does not need to be written to the hive files

You observe multiple Prefetch (.pf) files for SVCHOST.EXE, each with differing hashes. What does this indicate?

*

1 point

A new Prefetch file is created for SVCHOST.EXE upon system reboot

This is normal: multiple SVCHOST.EXE files exist in multiple locations on modern Windows systems

This is normal: the -k parameter is included in the hash computation for this particular process, which is why there are so many variations

The system is compromised and should be investigated immediately

What does the modification time of a Prefetch .pf file indicate?

*

1 point

The time the operating system was installed

The time that binary was created on the file system

The first time that particular binary executed, minus 0-10 seconds

The last time that particular binary executed, minus 0-10 seconds

On a Windows Server 2022 system, you notice the C:\Windows\Prefetch directory is empty. What could this indicate?

*

1 point

This is normal - Prefetch is not enabled by default on Windows Server operating systems

Someone has intentionally disabled Prefetch on this operating system

This is normal - Prefetch on Windows Server operating systems is only written on reboot or shutdown

Someone has performed anti-forensics and removed all Prefetch files

What timestamp is associated with Shimcache/AppCompatCache?

*

1 point

The Creation Time (B) of the binary

The time the binary was shimmed

There is no timestamp associated with Shimcache/AppCompcatCache

The Last Modification Time (M) of the binary

The time the binary was executed

When is Shimcache/AppCompatCache written to disk?

*

1 point

When the Windows Compatibility Telemetry Scheduled Task is run

Within 0-10 seconds after a binary has been shimmed

Hourly, when the SYSTEM Registry hive transaction journals are rolled into the hive

Every 24 hours

On reboot or shutdown only

Which of the following statements are true regarding Shimcache/AppCompatCache?

*

1 point

The binaries that are visible within an Windows Explorer file window can determine what enters the cache

Only binaries that are executed from the Command Prompt will be shimmed

Only binaries that are executed from the GUI will be shimmed

The binary must be selected within an Explorer window for a period of at least 10 seconds before it will be shimmed

Only binaries viewed from the Command Prompt will be shimmed

What is the primary reason an investigator would choose to use Shimcache/AppCompatCache?

*

1 point

The timestamp associated with Shimcache/AppCompatCache can be used to show earlier threat actor activity that may otherwise have gone unnoticed

Shimcache/AppCompatCache a reliable artifact that can be used to show program execution on Windows 10/11

Shimcache/AppCompatCache utilizes a Cache Entry Position to denote the number of times a given binary was executed

Shimcache/AppCompatCache can be used to show presence of a file that once existed on disk but may no longer be present

If two different binaries tracked within Shimcache/AppCompatCache have the exact same timestamp (even when comparing the full 64-bit Windows Filetime), what can be determined?

*

1 point

The two files executed at exactly the same time

The two files were copied at exactly the same time

That is not possible – Shimcache/AppCompatCache would not log two entries with an identical timestamp

The two files are likely the same, and by using the Cache Entry Position, the file with the lower position was likely the original name, and the file with the higher position (nearest the top of the parsed output) was likely the new name

What is the primary reason an investigator would choose to use AmCache?

*

1 point

AmCache is a valuable artifact that can be relied upon to show GUI-based program execution

AmCache a valuable source of metadata for program binaries, even if it can’t be used to definitively prove a given program executed

AmCache tracks execution timestamps for GUI or CLI-based programs

AmCache tracks the directories and files utilized by a given executable, providing insight into the capabilities of a given binary

Which of the following “Evidence of Execution” artifacts was introduced in Windows 11 22H2?

*

1 point

MUICache

ProgramCache

Superfetch

ApplicationExperience

Executable Assistant Updater (EAU)

Program Compatibility Assistant (PCA)

Which of the following statements are true regarding MUICache?

*

1 point

MUICache tracks the metadata associated with an executable along with the time it was executed

The LastWriteTimestamp of the MUICache key can be used to determine when the last value listed in MUICache was executed

MUICache is stored within each user’s NTUSER.DAT Registry Hive

MUICache does not use an MRU list, nor does it track the time a program was executed

Which of the following items are tracked by MUICache?

*

1 point

Only the CompanyName and FileDescription from the binary’s version info resource can be gleaned from this artifact

The full path and filename of the program that was executed and the CompanyName and FileDescription from the binary’s version info resource

The CompanyName and FileDescription from the binary’s version info resource and the total number of times the program executed

The CompanyName and FileDescription from the binary’s version info resource, the total number of times the program executed, and the last time of execution

Which of the following statements are true regarding UserAssist?

*

1 point

UserAssist tracks GUI and CLI-based program execution

UserAssist is only enabled on Windows desktop operating systems (not Windows Server)

UserAssist data is Base64 encoded

UserAssist tracks the total number of times a program was run by a specific user

UserAssist is a per-user program execution artifact stored within each user’s UsrClass.dat Registry Hive

Which of the following statements are true regarding the System Resource Utilization Monitor (SRUM)?

*

1 point

SRUM can be used as an “Evidence of Execution” artifact, and can also be used to determine how much data a given process transferred across the network

SRUM can be used to track the total number of times a program has been executed

SRUM is stored within the SYSTEM Registry Hive

SRUM can be used to profile user behavior on a Windows system, but cannot be used to determine what programs executed

Where is SRUM located?

*

1 point

It’s stored in its own Registry Hive call SRUM within %SystemRoot%\System32\config

It’s stored within the SYSTEM Registry Hive

It’s a database stored within %SystemRoot%\System32\LogFiles\Sum\Current.mdb

It’s a database located within %SystemRoot%\System32\sru\SRUDB.dat

What Evidence of Execution artifact also tracks files and directories referenced by a given executable?

*

1 point

MUICache

Shimcache

UserAssist

Prefetch

AmCache

Assuming an optimal logging configuration was in place to align with best practices, what Event Log(s) (Channels) and Event ID(s) could be reviewed to determine whether new Scheduled Tasks had been created on a system of interest?

*

1 point

Only Event ID 4698 within Security can be used to show Scheduled Task creation

Event ID 601 within Task-Scheduler

Only Event ID 106 within Microsoft-Windows-TaskScheduler/Operational can be used to show Scheduled Task creation

Event ID 4698 within Security; Event ID 106 within Microsoft-Windows-TaskScheduler/Operational

What effect would the following command line have:  
reg add HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest /v UseLogonCredential /t REG_DWORD /d 0

*

1 point

This sets WDigest to a secure (default) state, effectively disabling the protocol

This setting enables Windows Defender Credential Guard

This has no effect on WDigest, as “UseLogonCredential” is not the correct value name

This sets WDigest to an insecure state and enables storage of cleartext credentials within LSASS

Which of the following statements are true regarding User Access Logging (UAL)?

*

1 point

User Access Logging can be used to map lateral movement paths from endpoints to Windows Server-based operating systems

None of these statements regarding UAL is true

User Access Logging is disabled by default

User Access Logging is available in all modern Windows operating systems, and is enabled by default

Assume the System Event Log has been cleared. What could you look for within an environment to determine if a specific Windows Server-based operating system was the recipient of a remote connection via PsExec?

*

1 point

The creation of a file called PsExec.exe in C:\Windows\

The creation of HKEY_CURRENT_USER\Software\Sysinternals\PsExec with a EulaAccepted value of 1

Presence of a Prefetch .pf file for PSEXESVC.EXE

Shimcache entries referencing PSEXESVC.EXE

You observe Event ID 7045 within the System log referencing a service called “BTOBTO.” What remote execution/lateral movement tool was likely used against that system?

*

1 point

btexec.py

smbexec.py

atexec.py

wmiexec.py

psexec.exe

You observe the following command line within Event ID 4688 in the Security log:

cmd.exe /Q /c command 1> \\127.0.0.1\ADMIN$\__1679596932.6349123 2>&1

What remote execution/lateral movement tool was likely used against that system?

*

1 point

wmiexec.py

psexec.exe

Cobalt Strike

smbexec.py

btexec.py

You see the following empty directories created on a Domain Controller during a period of time in which a Threat Actor was known to have been active on the system: C:\ProgramData\backup\Active Directory and C:\ProgramData\backup\registry. What has likely happened?

*

1 point

Nothing – these directories exist by default on Windows Server-based operating systems with the AD DS role installed

The Threat Actor has likely staged an LSASS dump and exported the registry to the aforementioned path

The Threat Actor has likely used ntdsutil.exe to stage NTDS.dit and the SYSTEM and SECURITY registry hives

The Threat Actor has likely used esentutl.exe to stage NTDS.dit and the SYSTEM and SECURITY registry hives

The NTFS file reference number is:

*

1 point

Another name for the 2-byte MFT Sequence Number

A combination of the 6-byte MFT Entry Number and 2-byte MFT Sequence Number

A combination of the Log File Sequence Number and the Update Sequence Number

A combination of the Update Sequence Number and the Parent File Reference Number

Assuming the default NTFS cluster size for a disk < 16TB, how many clusters would be required to store a file that was 325,686 bytes.

*

1 point

20 clusters

79 clusters

21 clusters

This cannot be determined with the provided information

80 clusters

If a file’s Modification (M) timestamp was backdated and then the file was renamed such that the $FILE_NAME timestamp was updated to the same value, how could you detect this?

*

1 point

Look for TimeModOld, TimeModNew opcodes within the $J

Use cmd.exe and run "dir" against the file to see the original, non-backdated timestamp

Look for RenameOldName, RenameNewName opcodes within the $J

There is no known forensic method to detect this

What is the typical size of an MFT FILE record?

*

1 point

1024 bytes

2048 bytes

4096 bytes

There is no typical size as it can vary from one record to the next

In NTFS, what is a Resident File?

*

1 point

A file with an Alternate Data Stream (ADS) present

A file of 700-800 bytes or less that is small enough such that the file’s contents are stored within the MFT FILE record itself

A file with one or more extended attributes set

A file larger than 700-800 bytes such that one or more data runs is present within the MFT FILE record specifying the clusters occupied by that file

What is a Zone Identifier?

*

1 point

A component of the $UsnJrnl that tracks file downloads

A backwards compatibility mechanism to ensure interoperability between Windows NTFS and Apple HFS+

An Alternate Data Stream (ADS) that stores metadata including, in some cases, the location from which a file was downloaded

An NTFS metafile that centrally stores a list of all downloads and the URLs from which those downloads originated

What NTFS metafile could be leveraged to potentially determine if a file once existed in a given directory, even if that file has been securely wiped from the disk?

*

1 point

Look for an orphaned MFT FILE record referencing that file

$MftMirr could be leveraged to find evidence of the file’s existence

Slack space within $I30 for that particular directory

File carving

Which file within the Windows 10/11 Recycle Bin contains metadata for the file that was “recycled”, and which file contains the actual file contents?

*

1 point

Only the original file is moved to the Recycle Bin directory; the metadata is stored elsewhere

INFO2 contains the metadata, the original file will retain its name and be moved to the Recycle Bin directory

$I contains the original file, $R contains the metadata

$I contains the metadata, $R contains the original file

Which NTFS file system timestamp (MACB) is NOT exposed to the Windows API, and therefore NOT available for display within Windows Explorer?

*

1 point

MFT Record Change

Birth (Creation)

Modification

Access

From what location are the three (3) NTFS file system timestamps shown in Windows Explorer derived?

*

1 point

They are the $FILE_NAME timestamps from each file’s MFT FILE record

They are the $STANDARD_INFORMATION timestamps from each file’s MFT FILE record

They are pulled from the NTFS file system journals

They are pulled from the $FILE_NAME attribute associated with the $I30 index record for the directory in which the file resides

Is it possible to determine when a file was deleted on an NTFS file system?

*

1 point

Yes, look for a “File Delete” opcode associated with $J ($UsnJrnl)

Yes, look for the “D” timestamp within $FILE_NAME

Yes, the MFT FILE Record Flag at 0x16 is updated to indicate the time the file was deleted

No, the MACB timestamps remain intact upon a file’s deletion, and there is no way to determine the actual time of deletion

Which of the following statements are true regarding LNK files?

*

1 point

LNK files are only created for files present on the system volume (typically C:)

LNK files contain an “execute flag” that can be leveraged to determine when a particular program executed

LNK files are automatically created upon the creation of certain file types, and upon deletion of the original file, the LNK file is also immediately deleted

LNK files contain timestamps for the original file to which the LNK file pointed

Which of the following statements are true regarding Jump Lists?

*

1 point

Jump Lists are collections of LNK files (in one of two formats)

Jump Lists are stored within each user’s NTUSER.DAT registry hive

Jump Lists are only created for programs that are pinned to the Windows Taskbar

Jump Lists are used by Windows to automatically start programs on user logon

What is places.sqlite?

*

1 point

The main database containing history, bookmarks, and downloads for Google Chrome

The main database containing cache for Mozilla Firefox

The main database containing history, bookmarks, and downloads for Microsoft Edge

The main database containing history, bookmarks, and downloads for Mozilla Firefox

What web browser artifact tracks local file access in Windows?

*

1 point

Places.sqlite

The Windows Search database

WebCacheV01.dat

WordWheelQuery

Local file access is not tracked unless that file was opened with a browser (for example, a PDF opened in Microsoft Edge)

Besides “Default”, what other directories within %LocalAppData%\Microsoft\Edge\User Data\ could contain browser artifacts?

*

1 point

Profile 1, Profile 2, etc.

Default 1, Default 2, etc.

User 1, User 2, etc.

None – all information would be stored within “Default”

What do Microsoft Edge history entries prepended with file:/// indicate?

*

1 point

An offline/cached website

Files that were accessed via a UNC/network share

Files that were opened within Microsoft Edge (PDFs, for example)

Local file access

Will Thumbs.db still be present in Windows 10/11 systems?

*

1 point

Yes, Thumbs.db is still used for locations view in icon view residing on drives other than C:

Yes, Thumbs.db is still generated for hidden folders viewed in icon view

No, Thumbs.db was replaced with Thumbcache starting in Windows Vista

Yes, Thumbs.db is still used for locations accessed via a UNC/network path

What does the numerical value appended to Thumbcache_ files indicate?

*

1 point

The X (width) resolution of the thumbnail images contained within that database

The version of the thumbnail cache in use

The maximum thumbnail size for images contained within that database

The maximum number of thumbnail images contained within that database

Which of the following statements are true regarding thumbnails?

*

1 point

Thumbs.db files are centrally stored within each user’s profile

Thumbcache files are hidden databases created within any directory viewed in icon view via Windows Explorer

Thumbcache files are centrally stored within each user’s profile

Thumbs.db files are no longer utilized as of Windows 10

[PRACTICAL] What is the name of the computer system?  
*

2 points

CELTRIS

JEAN-LUC

DESKTOP-AG491CA

XSPACE2197

KATAAN

[PRACTICAL] In what time zone was this computer system?  
*

2 points

Coordinated Universal Time (UTC)

Mountain Standard Time

Central Daylight Time

Eastern Standard Time

Pacific Daylight Time

[PRACTICAL] What is the IP address assigned to this computer system?  
*

2 points

192.168.30.2

192.168.10.2

10.10.10.129

192.168.30.129

192.168.10.40

[PRACTICAL] What is the serial number of the USB flash drive connected to the system on 2023-03-24?  
*

2 points

6&30c5d09c&0&5

4C530001150702116201

6&35d1f50b&0&2

VID_0781&PID_5583

7&3ae26960&0&0000

[PRACTICAL] What is the first (oldest) command the Jean-Luc user typed into the Run prompt (Start > Run)?  
*

2 points

powershell

bca

ncpa.cpl

mstsc

cmd

[PRACTICAL] What is the name of the text file that once existed on the Desktop of the Jean-Luc user?  
*

2 points

downloads.txt

temp.txt

password.txt

updater.txt

creds.txt.txt

[PRACTICAL] What is the creation time (B) of the text file that once existed on the Desktop of the Jean-Luc user?  
*

2 points

2023-03-24 20:41:12

2023-03-24 18:45:10

2023-03-24 18:44:30

2023-03-24 18:54:59

2023-03-24 18:44:33

[PRACTICAL] What is the size of the text file that once existed on the Desktop of the Jean-Luc user?  
*

2 points

413 bytes

421 bytes

0 bytes

565 bytes

82 bytes

[PRACTICAL] What is the name of the malware present on this computer system?  
*

2 points

HackTool:Win32/Mimikatz.D

Trojan:Win32/Generic!rfn

Ransom:Win32/Royal.A!dha

Trojan:Win32/Wacatac.H!ml

[PRACTICAL] What is the full path of the file that triggered the malware detection?  
*

2 points

C:\Users\Jean-Luc\Downloads\iepv.exe

C:\Windows\System32\cmd.exe

C:\ProgramData\Microsoft\Windows Defender\Scans\MsMpEngCP.exe

C:\ProgramData\system\a.exe

C:\Windows\explorer.exe

[PRACTICAL] What is the IP address associated with the RDP reconnection events for the XSPACE2197\Jean-Luc user?

*

2 points

10.10.10.129

192.168.30.129

192.168.30.1

192.168.10.2

192.168.10.40

[PRACTICAL] How many times was this computer system the recipient of a PsExec session?

*

2 points

7

9

5

1

3

[PRACTICAL] What is the name of the system from which the PsExec session(s) originated?  
*

2 points

DESKTOP-AG491CA

JEAN-LUC

KATAAN

XSPACE2197

CELTRIS

[PRACTICAL] A USB flash drive was connected to this computer system and mounted as E:. What is the name of the folder that was browsed to on this volume?  
*

2 points

updater

ProgramData

Temp

backup

backup_20230324

[PRACTICAL] What is the name of the file located in the Jean-Luc user’s Downloads directory that is no longer present on disk?  
*

2 points

sdelete.exe

iepv.zip

imzadi.exe

Temp.exe

q.exe

[PRACTICAL] What is the original application name of the binary located in the Jean-Luc user’s Downloads directory that is no longer present on disk?

*

2 points

fsquirt

7-Zip

13Cubed

Secure file delete

APG

[PRACTICAL] What is the SHA-1 hash of the binary located in the Jean-Luc user’s Downloads directory that is no longer present on disk?  
*

2 points

82D19030F67F5FD14D2312FAF02A970E144E20DF

4AA4B498AE037A2B0479659374A5C3AF5F6B8D97

CA07DC6DF974BB1269957D52091F989B469BCBAD

5B3B21F15DF643DC43B6AEFE4E17F3929CBC43BE

438C3E852508ABDD8C6463317DADDDB87DB7FADF

[PRACTICAL] What is the original name and URL from which temp.pdf, located in C:\Users\Jean-Luc\Downloads, was obtained?  
*

2 points

[https://www.13cubed.com/downloads/rdp_flowchart.pdf](https://www.google.com/url?q=https://www.13cubed.com/downloads/rdp_flowchart.pdf&sa=D&source=editors&ust=1763210542390636&usg=AOvVaw1GD8s9z_4rgfGLIQ9cDRiH)

[https://visualstudio.microsoft.com/keyboard-shortcuts.pdf](https://www.google.com/url?q=https://visualstudio.microsoft.com/keyboard-shortcuts.pdf&sa=D&source=editors&ust=1763210542390818&usg=AOvVaw2sZpsQIAguAgGibjXwtwjk)

[https://www.13cubed.com/downloads/ntfs_file_record.pdf](https://www.google.com/url?q=https://www.13cubed.com/downloads/ntfs_file_record.pdf&sa=D&source=editors&ust=1763210542390921&usg=AOvVaw2tvqgq_zeotvB6DEvrGc-Z)

[https://www.13cubed.com/downloads/impacket_exec_commands_cheat_sheet_poster.pdf](https://www.google.com/url?q=https://www.13cubed.com/downloads/impacket_exec_commands_cheat_sheet_poster.pdf&sa=D&source=editors&ust=1763210542391027&usg=AOvVaw3mYhyXCqhyHRKGTZMjn61I)

[https://www.13cubed.com/downloads/windows_browser_artifacts_cheat_sheet.pdf](https://www.google.com/url?q=https://www.13cubed.com/downloads/windows_browser_artifacts_cheat_sheet.pdf&sa=D&source=editors&ust=1763210542391142&usg=AOvVaw2Uil33EVuyl8nwyFe7i-xF)

[PRACTICAL] What is the IP address of the remote host the Threat Actor connected to via a file transfer utility?  
*

2 points

52.73.116.225

192.168.10.40

192.168.10.50

10.0.0.40

10.0.0.50

[PRACTICAL] What is the full path of the binary specified within the Scheduled Task created by the Threat Actor?  
*

2 points

C:\Windows\System32\wuaucl.exe

C:\PerLogs\updater.exe

C:\ProgramData\Updater\update.exe

C:\ProgramData\a.exe

C:\PerfLogs\a.exe