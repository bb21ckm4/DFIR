
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
