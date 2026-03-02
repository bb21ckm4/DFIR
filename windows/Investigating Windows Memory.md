
---

# Welcome and Introduction

- An overview of the course and what to expect
    
- Personal notes about the development of this course
    
- Course layout and navigation
    

**💖 Special Thanks:**

- **Meaghan Bradshaw** for creation of the malware memory images used in this course
    
- **Ulf Frisk** for review of the MemProcFS lessons
    
- **Mark Spencer** from Arsenal Consulting for review of the Hibernation Files lesson

---

# Initial Setup

An overview of the Windows features and software we will use throughout this course. Learn how to configure your own system so that you can follow along.

---

**⚙️ Windows Features:**

**Windows Subsystem for Linux (WSL)**

Windows Subsystem for Linux (WSL) allows you to run a full Linux distribution directly within Windows, without needing a separate virtual machine. It provides access to familiar Linux tools and command line utilities, making it easier to analyze artifacts, parse logs, and perform scripting tasks alongside native Windows utilities.

1. Open PowerShell as Administrator.  
2. Type `wsl --install`.  
3. Restart when prompted. Ubuntu 24.04 LTS will be installed by default.  
4. Again, open PowerShell as Administrator.  
5. Type `wsl --install -d Ubuntu-22.04` to install Ubuntu 22.04 LTS, required for Volatility 2.

You can now use Windows Terminal to open separate tabs for each Ubuntu distro, allowing you to install and use Volatility 2 and Volatility 3 side by side.

Next, install the following tools before continuing with the course.

---

**🛠️ Tools:**

- [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager)
    
    - A trusted tool for acquiring forensic images, viewing disk contents, and exporting individual files or folders from evidence.
        
- [Velocidex WinPmem](https://github.com/Velocidex/WinPmem/releases/latest)
    
    - A tool for acquiring physical memory (RAM) from live Windows systems, preserving volatile data for later forensic analysis.
        
- [Magnet DumpIt for Windows](https://www.magnetforensics.com/resources/magnet-dumpit-for-windows/)
    
    - A tool for acquiring physical memory (RAM) from live Windows systems, preserving volatile data for later forensic analysis.
        
- [Belkasoft RAM Capturer](https://belkasoft.com/ram-capturer)
    
    - A tool for acquiring physical memory (RAM) from live Windows systems, preserving volatile data for later forensic analysis.
        
- Volatility 2 | Volatility 3
    
    - Open-source frameworks for memory forensics that support a wide range of analysis plugins for extracting processes, DLLs, network connections, and other artifacts from memory images.
        
        - **Ubuntu 22.04 is required for Volatility 2 and will also work for Volatility 3. However, newer Ubuntu releases, including the default WSL distro, can be used for Volatility 3 without issue.**
            
```bash
Volatility 2 Installation (*** Ubuntu 22.04 ONLY ***)

# Create and Change Into Tools Directory
mkdir ~/tools
cd ~/tools

# Install Volatility 2
sudo apt update && sudo apt full-upgrade -y
sudo apt install -y build-essential python2 python2-dev
wget https://bootstrap.pypa.io/pip/2.7/get-pip.py
sudo python2 get-pip.py
pip2 install pycrypto distorm3==3.4.4 yara-python==4.0.0
git clone https://github.com/volatilityfoundation/volatility.git
cd volatility

# Verify Successful Installation
python2 vol.py --info | more

```

```bash
Volatility 3 Installation (*** Ubuntu 24.04 and later ***)

# Change Into Tools Directory
cd ~/tools

# Install Volatility 3
sudo apt update
sudo apt install -y python3-pip python3-pefile python3-yara

## Install pycrypto (Ubuntu 22.04 ONLY)
pip3 install pycrypto

## Install pycryptodome (Ubuntu 24.04 ONLY)
## Note that --break-system-packages is necessary to bypass restrictions on non-Debian Python packages.
pip3 install pycryptodome --break-system-packages

git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3

# Verify Successful Installation
python3 vol.py -h | more

```


- [MemProcFS](https://github.com/ufrisk/MemProcFS/releases/latest)
    
    - A live memory analysis framework that mounts a memory image as a virtual file system, allowing easy access to processes, modules, and other artifacts.
        
- [Dokany (User Mode File System Library)](https://github.com/dokan-dev/dokany/releases/latest)
    
    - A required library for MemProcFS that enables file system functionality in user mode, allowing memory images to be mounted as virtual file systems.
        
- [WinDbg](https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/)
    
    - Microsoft’s advanced debugger for analyzing crash dumps, kernel memory, and live systems, offering deep insight into Windows internals.
        
- [Hibernation Recon](https://arsenalrecon.com/downloads)
    
    - A specialized tool for parsing and extracting data from Windows hibernation files (`hiberfil.sys`), useful for recovering memory-resident artifacts.
        
- [Eric Zimmerman's Tools](https://ericzimmerman.github.io/)
    
    - A collection of powerful Windows forensic utilities for parsing event logs, registry hives, prefetch files, and many other artifacts.
        

---

**There are five (5) primary memory images used within this course. See below for a list of the modules within which each image is used. Each lesson's description within those modules will also contain a link to the applicable image.**

**📦 Memory Images:**

- [**win10_clean.zip**](https://files.13cubed.com/iwm/win10_clean.zip)
    
    - Memory Analysis with Volatility
        
    - Memory Analysis with MemProcFS
        
    - Introduction to WinDbg
        

- [**win11_infected.zip**](https://files.13cubed.com/iwm/win11_infected.zip)
    
    - Malware Memory Analysis with Volatility
        
    - Malware Memory Analysis with MemProcFS
        

- [**practice_crackmapexec+metasploit.zip**](https://files.13cubed.com/iwm/practice_crackmapexec%2Bmetasploit.zip)
    
    - N/A (Practice Memory Image)
        

- [**practice_powershell_empire.zip**](https://files.13cubed.com/iwm/practice_powershell_empire.zip)
    
    - N/A (Practice Memory Image)
        

- [**image.zip**](https://files.13cubed.com/iwm/image.zip)
    
    - Knowledge Assessment
        

Chris Burns

Jul 18, 2024

Just in case anyone has the same issue I did when trying to execute the following:

```
pip2 install pycrypto distorm3==3.4.4
```

I had to run the following command to install some dependencies and allow pycrypto to successfully install:

```
sudo apt-get install build-essential libssl-dev libffi-dev python2-dev
```

Hopefully this saves anyone else having to search through stack overflow for a solution :')

---

# Foundations of Memory Forensics

---

## Windows Memory Structures

A discussion about memory forensics and why it's still very relevant today. An overview of Windows memory structures, including the Kernel Debugger Data Block (KDBG), _EPROCESS blocks, the Process Environment Block (PEB), Virtual Address Descriptor (VAD) trees, and more.

**Resources:**  
[Windows Memory Structures](https://cdn.13cubed.com/downloads/windows_memory_structures.pdf)

---

## Windows Process Genealogy

An overview of the core Windows system processes. Learn what "normal" looks like so that you can quickly find anomalies.

**Resources:**  
[Windows Process Genealogy](https://cdn.13cubed.com/downloads/windows_process_genealogy.pdf)

---

# Acquiring Memory

---

## The Basics

Learn the basics of memory acquisition, memory smearing, and the differences between Windows Crash Dump and RAW formats.

---

## The Tools

Learn about the available tools for acquiring memory from live computer systems in RAW or Windows Crash Dump formats.

---

## Best Practices for Virtual Machines

Learn about best practices regarding memory acquisition from virtual machines, including the differences between acquisition via suspend versus snapshot.

---

## VMware ESXi

Learn how to acquire memory from VMware ESXi guest virtual machines.

**If you would like to use the free version of VMware ESXi, follow the links below to create a Broadcom account and download the software.**

- [Broadcom Support Portal: User Registration (Required to Download)](https://profile.broadcom.com/web/registration)
    
- [VMware vSphere Hypervisor (Free)](https://support.broadcom.com/group/ecx/productdownloads?subfamily=VMware%20vSphere%20Hypervisor&freeDownloads=true)
    

**Resources:**  
[Memory Forensics for Virtualized Hosts](https://blogs.vmware.com/security/2021/03/memory-forensics-for-virtualized-hosts.html)  
[Proxmox VE](https://www.proxmox.com/en/downloads/proxmox-virtual-environment)

- Make sure the .vmsn and .vmem file have the same name and are in the same location when downloaded.  Volatility 3 uses the vmsn file to map memory locations so without it this wont work and MemProcFS is the same way
---

## Microsoft Hyper-V

Learn how to acquire memory from Microsoft Hyper-V guest virtual machines.

**Resources:**  
[Sysinternals LiveKd](https://learn.microsoft.com/en-us/sysinternals/downloads/livekd)  
[Windows SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-sdk/)

**ℹ️ Alternative Solution Using MemProcFS:**  
As mentioned in this lesson, MemProcFS also supports Hyper-V checkpoints and can be used as an alternative to Sysinternals LiveKd. See [MemProcFS Documentation](https://github.com/ufrisk/LeechCore/wiki/Device_HyperV_SavedState) for more information.

---

# Poor Man's Memory Forensics

---

## Strings and Bstrings

Learn how to use `strings` and `bstrings` to perform basic analysis of Windows memory images.

I always recommend using `bstrings` instead of `strings` (at least initially) for one simple reason: by default, bstrings looks for both ASCII and Unicode strings. `strings` (on Linux) requires you to choose the Unicode encoding via `-e l`, with the default option to only look for single-7-bit-byte characters (e.g., ASCII). There's no option to search for both encodings at once.

To confuse matters further, the Sysinternals suite version of `strings` does not suffer this issue -- it also looks for both Unicode and ASCII strings by default.

---

## Pagefile.sys and Swapfile.sys

Learn about `pagefile.sys` and `swapfile.sys`, their purpose, and how to analyze them.

-Volatility doesn't work on pagefile.sys.
-You may get malware signatures due to AV in pagefile.sys so you will probably find alot of false positives.  Searching for cobalt strike or mimikatz etc..
-There is a great PS script "MemProcFS-Analyzer' I use generally for memory analysis aside of Volatility. I found a tons of indicators of different ransomwares, cobalt strike etc. and I was completely shocked, and was determined it was the result from 2 AV engines.

---

# Memory Analysis with Volatility

---

## Image Identification and Metadata

Learn how to perform basic identification of a memory image and extract important metadata using `imageinfo`, `kdbgscan`, and `windows.info`.

**⚠️ Important:**  
Ensure that when you unzip win10_clean.zip you place both files, win10_clean.vmem and win10_clean.vmsn, in the same location. The VMSN file must be present alongside the VMEM file in order for our memory forensics tools to be able to properly read the image.

**ℹ️ Tip:**  
While not used in this course, note that **Volatility 3** offers a `--save-config` option and a `-r pretty` formatting option that may be useful to you:

- The former option will allow you to save metadata for the memory image to speed up subsequent execution of plugins (recall the saved config with `-c`).
    
- At several points in the following lessons, I mention that Volatility 3 plugin output is often not as readable as Volatility 2. The latter option will partially address that and provide a cleaner output for many of the plugins to make it easier to interpret the results.
    

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Basic Process Enumeration - Part 1

Learn how to enumerate processes using `pslist` and `windows.pslist`.

Note that Volatility 3 now supports specifying multiple PIDs with the `--pid` flag, using space separators.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Basic Process Enumeration - Part 2

Learn how to enumerate processes using `pstree` and `windows.pstree`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## In-depth Process Enumeration

Learn how to enumerate processes via pool tag scanning to find potentially exited or unlinked processes using `psscan` and `windows.psscan`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Comparison of Process Enumeration Methods

Learn about manual options to compare the output of process-related plugins, then see how the legacy Volatility 2 plugin `psxview` automates those comparisons, and how the same task can now be accomplished in Volatility 3 with the `windows.malware.psxview` plugin.

📦 **Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Dynamic Link Libraries (DLLs)

Learn how to obtain a list of Dynamic Link Libraries (DLLs) associated with processes using `dlllist` and `windows.dlllist`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Process Command Lines

Learn how to obtain the full command line associated with a process using `cmdline` and `windows.cmdline`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Process Handles

Learn how to obtain handles (file, registry, etc.) associated with a process using `handles` and `windows.handles`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Process Security Tokens

Learn how to identify the security context of a process via enumeration of its tokens using `getsids` and `windows.getsids`.

[https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/understand-security-identifiers)

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Network Activity

Learn how to enumerate network activity using `netscan`, `windows.netscan`, and `windows.netstat`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

**Resources:**  
[Abeebus - GeoIP Lookup Utility](https://github.com/13Cubed/Abeebus/releases/latest)

In November 2023, I released [**SharpAbeebus**](https://github.com/13Cubed/SharpAbeebus/releases/latest) - a modern C# .NET 8 rewrite of the original Python 3 Abeebus GeoIP lookup utility shown in this lesson. It works the same way but is much faster and more efficient! Please consider using it instead.

---

## Registry Analysis

Learn how to list registry hives, print registry keys, subkeys, and values, and extract other registry-related artifacts using:

- `hivelist` and `windows.registry.hivelist`
    
- `hivescan` and `windows.registry.hivescan`
    
- `printkey` and `windows.registry.printkey`
    
- `userassist` and `windows.registry.userassist`
    

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

**Resources:**  
[Windows Registry Cheat Sheet](https://cdn.13cubed.com/downloads/windows_registry_cheat_sheet.pdf)

---

## Basic Code Injection

Learn how basic code injection works, and how to identify this activity using `ldrmodules` and `windows.ldrmodules` / `windows.malware.ldrmodules`.

⚠️ **Plugin Rename:**  
`windows.ldrmodules` is now `windows.malware.ldrmodules`  
Behavior and output are unchanged. This lesson uses the original name.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Reflective Code Injection

Learn how reflective code injection works and how it differs from basic code injection. Then, learn how to identify this activity using `malfind` and `windows.malfind` / `windows.malware.malfind`.

⚠️ **Plugin Rename:**  
`windows.malfind` is now `windows.malware.malfind`  
Behavior and output are unchanged. This lesson uses the original name.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Process Hollowing

Learn how process hollowing works, and how to identify this activity using `hollowfind` and `windows.hollowprocesses` / `windows.malware.hollowprocesses`.

⚠️ **Plugin Rename:**  
`windows.hollowprocesses` is now `windows.malware.hollowprocesses`  
Behavior and output are unchanged. This lesson uses the original name.

**Source:**  
[Detecting Deceptive Process Hollowing Techniques Using HollowFind Volatility Plugin](https://cysinfo.com/detecting-deceptive-hollowing-techniques/)  
Monnappa K A

**📦 Memory Images:**  
Stuxnet*  
[hollow.zip](https://files.13cubed.com/iwm/hollow.zip)

_*I do not own the rights to the Stuxnet memory image used in this lesson. The original download location is no longer available, but other copies are available via your search engine of choice._

**Resources:**  
[HollowFind Plugin](https://github.com/monnappa22/HollowFind)  
[Process Hollowing](https://github.com/m0n0ph1/Process-Hollowing)

Special thanks to Mike Peterson of [nullsec.us](http://nullsec.us/) for research and testing.  
  
🎉 **In April 2025,** [**ProcSentinel**](https://github.com/DFIROPS/-volatility3-plugins)**, a new third-party Volatility 3 plugin, was released to detect process hollowing and other memory-based anomalies. Our testing has shown it to be very promising, and we recommend adding this plugin to your library as well.**

If anyone's interesented in a nice organized output I found "--html-report " rather useful.

![](https://s3.us-west-2.amazonaws.com/content.podia.com/7k76i6v3zjnle9myy1fljl63vjf2)

---

## API Hooks

Learn how Application Programming Interface (API) hooks (also known as user-mode hooks) work, and how to find them using `apihooks`.

**📦 Memory Image:**  
Stuxnet*

_*I do not own the rights to the Stuxnet memory image used in this lesson. The original download location is no longer available, but other copies are available via your search engine of choice._

if interested, I found a link to the Dossier mentioned.  
[https://pax0r.com/hh/stuxnet/Symantec-Stuxnet-Update-Feb-2011.pdf](https://pax0r.com/hh/stuxnet/Symantec-Stuxnet-Update-Feb-2011.pdf)


Taking the opportunity to share this article by Lina (@inversecos) about Windows Event Log Evasion via Native APIs by Stuxnet.

In my opinion, it's a really good read as it talks about Volatility and memory internals (_EPROCESS, PEB, VAD, ...). More precisely, it shows how the technique works and why the "svcscan" plugin doesn't detect it. In the last part, she gives a great detection methodology (event logs, specific registry keys, ShimCache, Prefetch, $MFT and so on...)

Definitely worth looking at. :)

[https://www.inversecos.com/2022/03/windows-event-log-evasion-via-native.html](https://www.inversecos.com/2022/03/windows-event-log-evasion-via-native.html)


---

## SSDT Hooks

Learn how System Service Descriptor Table (SSDT) hooks (also known as kernel-mode hooks) work, and how to find them using `ssdt` and `windows.ssdt`.

**📦 Memory Image:**  
Stuxnet*

_*I do not own the rights to the Stuxnet memory image used in this lesson. The original download location is no longer available, but other copies are available via your search engine of choice._

I found the problem. Something broke in the very newest build of Volatility 3. It appears to have something to do with the automatic download of the symbols. Instead, try using the newest release version of Volatility 3, which is 2.7.0 ([https://github.com/volatilityfoundation/volatility3/releases](https://github.com/volatilityfoundation/volatility3/releases)).

Do the following in WSL:

- Type `cd ~/tools` (assuming that's where you installed Volatility)
    
- Type `wget https://github.com/volatilityfoundation/volatility3/archive/refs/tags/v2.7.0.tar.gz`
    
- Type `tar zxvf v2.7.0.tar.gz`, which will decompress this into ~/tools/volatility3-2.7.0
    
- Type `cd volatility3-2.7.0`, and try running `python3 vol.py /path/to/image windows.ssdt`

---

## Kernel Module (Driver) Enumeration

Learn how to enumerate kernel modules (also known as "drivers" in Windows-based operating systems) using:

- `modules` and `windows.modules`
    
- `modscan` and `windows.modscan`
    

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Dumping Files

Learn how to search for and dump (extract) files from memory using:

- `filescan` and `windows.filescan`
    
- `dumpfiles` and `windows.dumpfiles`
    

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Dumping Processes

Learn how to dump (extract) process binaries from memory using `procdump` and `windows.pslist` / `windows.psscan` with the `--dump` option.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Dumping Memory Sections

Learn how to dump (extract) process memory sections using `memdump`, `vaddump`, `memmap`, and `windows.memmap`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Dumping DLLs and Kernel Modules

Learn how to dump (extract) DLLs and kernel modules (drivers) using:

- `dlldump` and `moddump`
    
- `windows.dlllist` and `windows.modules` with the `--dump` option
    

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## YARA Scans

Learn how to perform in-memory YARA scanning using `yarascan`, `windows.vadyarascan`, and `yarascan.YaraScan`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Strings

Learn how to integrate strings output with Volatility to locate strings of interest within a memory image using `strings` and `windows.strings`.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Volatility Shell (volshell)

Learn how to interactively explore a memory image using the powerful `volshell` (Volatility Shell) command line interface.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

# Malware Memory Analysis with Volatility

---

## Inbrief

You've been hired by X-Space Corporation as a consulting forensic investigator. Watch this inbrief to learn the details of your assignment!

**⚠️ Important:**  
Ensure that when you unzip win11_infected.zip you place both files, win11_infected.vmem and win11_infected.vmsn, in the same location. The VMSN file must be present alongside the VMEM file in order for our memory forensics tools to be able to properly read the image.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

---

## Analysis - Part 1

Part 1 of 2 covering analysis of the memory image received from X-Space Corporation.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

**Resources:**  
[Immersive Labs Volatility 3 Plugin (13Cubed Fork)](https://raw.githubusercontent.com/13Cubed/windows.cobaltstrike/refs/heads/main/cobaltstrike.py)  
[Elastic YARA Rules](https://www.elastic.co/blog/detecting-cobalt-strike-with-memory-signatures)

ℹ️ **Plugin Update:**  
In November 2025, we forked and updated the Immersive Labs Volatility 3 plugin to support newer releases of Volatility 3. The link above points to the updated version.

---

## Analysis - Part 2

Part 2 of 2 covering analysis of the memory image received from X-Space Corporation.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

---

## Recap

This is the recap for the forensic investigation requested on behalf of X-Space Corporation.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

---

# Memory Analysis with MemProcFS

---

## Introduction

An introduction to MemProcFS.

**⚠️ Important:**  
Ensure that when you unzip win10_clean.zip you place both files, win10_clean.vmem and win10_clean.vmsn, in the same location. The VMSN file must be present alongside the VMEM file in order for our memory forensics tools to be able to properly read the image.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Running MemProcFS

---

Learn how to run MemProcFS and take full advantage of its numerous features and options.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Analysis

Learn how to analyze a memory image using MemProcFS.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

# Malware Memory Analysis with MemProcFS

---

## Running MemProcFS

Initial preparation for analyzing our malware memory image with MemProcFS.

**⚠️ Important:**  
Ensure that when you unzip win11_infected.zip you place both files, win11_infected.vmem and win11_infected.vmsn, in the same location. The VMSN file must be present alongside the VMEM file in order for our memory forensics tools to be able to properly read the image.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

---

## Finding Evil

Analysis of the memory image received from X-Space Corporation using MemProcFS.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

---

## Dumping Files

Learn how to dump (extract) files of interest from the memory image received from X-Space Corporation using MemProcFS.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

---

## Finding Files of Interest with WSL 2

Learn how to use MemProcFS with WSL 2 to locate files of interest within the memory image received from X-Space Corporation.

**📦 Memory Image:**  
[win11_infected.zip](https://files.13cubed.com/iwm/win11_infected.zip)

---

# Introduction to WinDbg

---

## What is WinDbg?

Learn the purpose of WinDbg and its place in memory forensics.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Acquiring a Windows Crash Dump with MemProcFS

Learn how to use MemProcFS to mount our clean memory image and obtain a copy of it in Windows Crash Dump format. This is the format required for WinDbg.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)

---

## Analysis

Learn how to analyze a memory image using WinDbg. Towards the end of this lesson, we'll also apply our knowledge of Windows memory structures and the circular doubly linked list to manually extract a list of processes from the image.

**📦 Memory Image:**  
[win10_clean.zip](https://files.13cubed.com/iwm/win10_clean.zip)*

_*See previous lesson for information on converting win10_clean.vmem to Windows Crash Dump format._ **_This is a prerequisite for this lesson._**

---

# Additional Content

---

## Hibernation Files

Learn about Windows Hibernation and hiberfil.sys. We'll discuss how this feature relates to memory forensics, and how to analyze hiberfil.sys with Volatility and MemProcFS.

**Resources:**  
[Windows Hibernation Infographic](https://arsenalrecon.com/insights/windows-hibernation-infographic)

---

## Shimcache Memory Forensics

Learn how to parse Shimcache from memory using the Volatility 3 `windows.shimcachemem` plugin. This information can serve as a valuable resource for providing evidence of program presence or existence.

---

## Additional Volatility 3 Plugins

This course has covered the core and most important Volatility 3 plugins used in Windows memory analysis. However, additional plugins are typically introduced with each new release, some ported from Volatility 2 and others entirely new. While not essential for every investigation, these plugins provide extended capabilities that may be useful in more specialized scenarios. Below is an alphabetized list of these plugins, along with a brief description of each.

**📅 Updated January 2026**

---

`windows.cmdscan`:  
Extracts command‑history buffers from cmd.exe processes.

`windows.consoles`:  
Recovers input/output data from Windows console buffers.

`windows.debugregisters`:  
Displays CPU debug‑register values for every thread.

`windows.deskscan`:  
Scans memory for Desktop objects that may be unreferenced.

`windows.desktops`:  
Enumerates all Desktop objects under each Window Station.

`windows.devicetree`:  
Enumerates the kernel device‑to‑driver tree.

`windows.driverirp`:  
Lists driver IRP (I/O Request Packet) hooks.

`windows.etwpatch`:  
Identifies ETW (Event Tracing for Windows) patching techniques used by malware to evade detection.

`windows.iat`:  
Examines Import Address Tables to detect hooking or stomping.

`windows.joblinks`:  
Lists job objects and their associated processes.

`windows.kpcrs`:  
Dumps the Kernel Processor Control Region for each CPU.

`windows.malware.direct_system_calls`:  
Detects direct system‑call techniques used to bypass user‑mode hooks (**previously** `windows.direct_system_calls`).

`windows.malware.drivermodule`:  
Flags hidden or tampered kernel drivers (**previously** `windows.drivermodule`).

`windows.malware.indirect_system_calls`:  
Detects indirect system‑call techniques used to bypass user‑mode hooks (**previously** `windows.indirect_system_calls`).

`windows.malware.pebmasquerade`:  
Detects potential process name spoofing by comparing EPROCESS and PEB data.

`windows.malware.processghosting`:  
Detects process‑ghosting anti‑forensics (**previously** `windows.processghosting`).

`windows.malware.suspicious_threads`:  
Flags threads with suspicious start addresses or APIs (**previously** `windows.suspicious_threads`).

`windows.malware.svcdiff`:  
Compares service lists to find hidden services (**previously** `windows.svcdiff`).

`windows.malware.unhooked_system_calls`:  
Detects processes where user‑mode EDR hooks were removed (**previously** `windows.unhooked_system_calls`).

`windows.mbrscan`:  
Scans for and parses Master Boot Records for malware.

`windows.mftscan.ADS`:  
Locates Alternate Data Streams inside carved FILE records.

`windows.mftscan.MFTScan`:  
Carves $MFT FILE records directly from memory.

`windows.orphan_kernel_threads`:  
Finds kernel threads without an owning process.

`windows.pe_symbols`:  
Extracts symbol information from in‑memory PE images.

`windows.pedump`:  
Dumps PE files from memory for static analysis.

`windows.registry.amcache`:  
Parses the Amcache hive for program‑execution evidence (**previously** `windows.amcache`).

`windows.registry.getcellroutine`:  
Reports hives whose GetCellRoutine pointer is hooked.

`windows.registry.scheduled_tasks`:  
Decodes Scheduled Task data from the registry (**previously** `windows.scheduled_tasks`).

`windows.sessions`:  
Lists interactive, service, and RDP sessions present in memory.

`windows.suspended_threads`:  
Enumerates threads currently in a suspended state.

`windows.svclist`:  
Enumerates registered Windows services via services.exe lists.

`windows.thrdscan`:  
Pool‑scans for hidden or unlinked thread objects.

`windows.threads`:  
Enumerates all threads via linked‑list traversal.

`windows.timers`:  
Displays kernel timers and their callback routines.

`windows.truecrypt.Passphrase`:  
Recovers cached TrueCrypt passphrases (**previously** `windows.truecrypt`).

`windows.unloadedmodules`:  
Lists recently unloaded kernel modules.

`windows.vadregexscan`:  
Runs regular‑expression searches across all VAD ranges.

`windows.vadwalk`:  
Traverses each process’s VAD tree for mapping analysis.

`windows.windows`:  
Enumerates top‑level window objects (HWND) for every Desktop.

`windows.windowstations`:  
Enumerates Window Station objects and their security descriptors.

---

## Practice Memory Images

Thank you for enrolling in Investigating Windows Memory! I hope you've enjoyed the course. You'll find links below for two (2) additional memory images that you can use for practice and to further your Windows memory analysis skills.

**📦 Practice Memory Images:**  
[practice_crackmapexec+metasploit.zip](https://files.13cubed.com/iwm/practice_crackmapexec%2Bmetasploit.zip)  
[practice_powershell_empire.zip](https://files.13cubed.com/iwm/practice_powershell_empire.zip)

---

## Trouble at ACME

![Trouble at ACME](https://s3.us-west-2.amazonaws.com/content.podia.com/4qsiad5b66uteops1yhmwh8tph0f)

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

## Chaos at Cobalt

![Chaos at Cobalt](https://s3.us-west-2.amazonaws.com/content.podia.com/20euzivflyhqqv98jm5fcdxedqik)

**Chaos at Cobalt**

This challenge is comprised of four (4) disk and four (4) memory images, and as an enrolled student, you now have exclusive access to both the challenge documentation (PDF) and the accompanying images.

You'll be tasked with investigating a compromise that took place at Cobalt Edge Technologies, impacting both Windows and Linux systems. The PDF provides all the initial details required to kickstart your investigation. This challenge serves as an opportunity for self-assessment, allowing you to gauge your forensic abilities across multiple platforms. Once you've reached the "**Stop Here!**" page, you can begin your investigation.

After completing your analysis and constructing a timeline of your findings, you'll have the chance to compare your results with the remainder of the document to evaluate your performance. It's important to note that, like real-world scenarios, uncovering 100% of the story may not always be feasible.

**To get started, see the "Chaos at Cobalt" PDF file at the link below.**

**📦 Downloads:**

- [chaos_at_cobalt.pdf](https://files.13cubed.com/iwe/cobalt/chaos_at_cobalt.pdf)
    
- [evidence_preparation.txt](https://files.13cubed.com/iwe/cobalt/evidence_preparation.txt)
    
- [cobalt1.zip](https://files.13cubed.com/iwe/cobalt/cobalt1.zip)
    
- [cobalt2.zip](https://files.13cubed.com/iwe/cobalt/cobalt2.zip)
    
- [cobalt3.zip](https://files.13cubed.com/iwe/cobalt/cobalt3.zip)
    
- [cobalt4.zip](https://files.13cubed.com/iwe/cobalt/cobalt4.zip)

---

