
---

# Welcome and Introduction

- An overview of the course and what to expect
    
- Personal notes about the development of this course
    
- Course layout and navigation
    

🚀 **About macOS 26 (“Tahoe”):**

- We have been working hands-on with macOS 26 since the initial Developer Beta
    
- No significant changes have been observed that impact forensic investigation
    
- All tools used in this course have been validated and confirmed to work under this release
    

**💖 Special Thanks:**

- **Mike Peterson** of [**nullsec.us**](http://nullsec.us/) for course content review
    
- **Meaghan Bradshaw** for creation of the course capstone challenge

---

# Initial Setup

An overview of the hardware and software requirements necessary to follow along with this course.

---

**💻 Required Hardware:**

- Apple Silicon-based Mac
    
- 16GB of RAM (Recommended)
    
- 250GB of Free Disk Space
    
- Broadband Internet (Recommended)
    

---

💾 **Required Software:**

- [Homebrew](https://brew.sh/)  
    Homebrew is a macOS package manager that we'll use to install various tools, including The Sleuth Kit (TSK) in later modules. To install it, open the macOS Terminal and paste the following line:  
    `/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`
    

⭐️ **Optional Software:**

- [Mactracker](https://mactracker.ca/)  
    Mactracker provides detailed information on every version of macOS and is used in the "History" lesson within the "Introduction to macOS" module.
    
- [Parallels Desktop](https://www.parallels.com/products/desktop/)  
    If you want to run a macOS virtual machine on your Mac, you will need to use Parallels Desktop. As of June 2025, VMware Fusion does not offer this capability. Note that Parallels Desktop is a paid product, but a 14-day free trial is available.
    

**➕ Additional tools will be installed as needed in later lessons.**

---

# Introduction to macOS

---

## History

A brief history of macOS and its evolution highlights the critical security and functionality improvements that have shaped the operating system. Understanding these changes is essential for appreciating their direct impact on forensic investigations of macOS devices, making this background knowledge incredibly valuable.

---

## Root Directory Structure

Learn about the purpose of all the common directories in the root of a macOS file system.

---

## File and Directory Permissions

Learn how to interpret macOS file and directory permissions. This lesson covers the `rwx` permission model, file flags, special permissions including the Sticky Bit, Set User ID (setuid), and Set Group ID (setgid), Access Control Lists (ACLs), and Extended Attributes (xattrs). By mastering these foundational concepts, you’ll gain skills needed to extract valuable forensic insights from macOS systems.

**Resources:**  
[macOS Extended Attributes Cheat Sheet](https://cdn.13cubed.com/downloads/macos_extended_attributes_cheat_sheet.pdf)

---

## Users and Groups

Learn how macOS manages users and groups, starting with an overview of account types and tools like `sysadminctl` and `dscl` for user and group management. Discover how macOS stores and secures passwords, including the per-user plist files that contain account password hashes. Understand the macOS Keychain, along with the Keychain Access and Passwords apps, with insights into how these tools store credentials, their encryption mechanisms, and techniques for accessing stored passwords. This lesson also addresses key forensic considerations, including system logs, hidden accounts, Secure Token, and Mobile/Network accounts.

---

**To extract password hashes on modern macOS systems, the following steps can be used:**

1. Copy the user plist to your user's Desktop:
```bash
$ sudo cp /var/db/dslocal/nodes/Default/users/<username>.plist ~/Desktop/
```
   
2. Change the permissions to 660 to allow further manipulation of the file:
```bash
$ sudo chmod 660 <username>.plist
```
   
3. Convert the binary plist to XML:
```bash
$ plutil -convert xml1 ~/Desktop/<username>.plist   
```
   
4. Extract the `ShadowHashData` field from the user’s plist file using a text editor, save it to a new file named `ShadowHashData.xml`, and decode the Base64 content into a binary plist for further analysis:
```bash
$ cat ShadowHashData.xml | base64 -d > ShadowHashData.bin
``` 
   
5. Verify the resultant file has the binary plist (`bplist00`) header.
  
**Format for Cracking**: After extracting the necessary components—**iterations**, **salt**, and **password hash**—from the `ShadowHashData`, you need to combine them into a specific format required by password-cracking tools like **Hashcat** or **John the Ripper**. This involves converting the extracted data (e.g., salt and hash) into hexadecimal and structuring it according to the PBKDF2-SHA512 format:

```bash
$pbkdf2-hmac-sha512$<iterations>$<salt>$<hash>
```

---

## Shells and Command History

A look at the most prevalent macOS shells you're likely to encounter in your investigations: the Bourne Again Shell (Bash), and Z Shell (Zsh). Learn about command history, configuration files, environment variables, and other important details.

---

## System Integrity Protection (SIP)

Learn how System Integrity Protection (SIP) enhances macOS security by restricting access to critical system files and limiting kernel extensions. In this lesson, we’ll explore SIP’s key features, understand its role in maintaining system integrity, and examine how it impacts forensic investigations. You’ll see how to check the status of SIP, learn the proper methods for temporarily disabling it (when necessary), and discover best practices for documenting any changes made.

---

## Transparency, Consent, and Control (TCC)

Learn how macOS’s Transparency, Consent, and Control (TCC) framework helps forensic investigators examine and trace access to critical system resources—from microphone and camera to Full Disk Access. This lesson shows you where the TCC databases reside, how to query them at both the system and user levels, and how these records can reveal essential evidence of unauthorized or suspicious resource usage.

---

## **Useful TCC Queries**

Ensure the Full Disk Access permission has been granted to your terminal or forensic tool before proceeding.

**System-level:**  
`sqlite3 "/Library/Application Support/com.apple.TCC/TCC.db" "PRAGMA table_info(access);"`

**User-level:**  
`sqlite3 ~"/Library/Application Support/com.apple.TCC/TCC.db" "PRAGMA table_info(access);"`

Look at the output to see if columns such as `timestamp`, `last_modified`, or others are present. Once you know the actual column names, you can adjust your queries accordingly.

**Queries for System-Level Permissions (Full Disk Access and other system-wide settings):**

**View all entries (system-wide):**  
`sqlite3 "/Library/Application Support/com.apple.TCC/TCC.db" "SELECT * FROM access;"`

In newer macOS versions, the `auth_value` in the TCC database commonly uses `0` for “Not Determined,” `1` for “Denied,” and `2` for “Allowed.” However, in older releases, these values may differ (for example, some older systems used `1` for “Allowed”). Always confirm what each value means by examining the actual data in your TCC.db.

**Identify applications granted Full Disk Access:**  
`sqlite3 "/Library/Application Support/com.apple.TCC/TCC.db" "SELECT client, auth_value, last_modified FROM access WHERE service='kTCCServiceSystemPolicyAllFiles' AND auth_value=2;"`

This command focuses on the service name `kTCCServiceSystemPolicyAllFiles`, which represents Full Disk Access permissions. By examining which applications hold this powerful permission, you can gain further insight into the system’s security posture and potential data exposure points.

**Queries for User-Level Permissions (microphone, camera, screen recording, etc.):**

**View all entries (per-user):**  
`sqlite3 ~"/Library/Application Support/com.apple.TCC/TCC.db" "SELECT * FROM access;"`

**List applications, their requested services, and access status (substitute** `timestamp` **or** `last_modified` **as appropriate):**  
`sqlite3 ~"/Library/Application Support/com.apple.TCC/TCC.db" "SELECT client, service, auth_value, last_modified FROM access;"`

**Identify applications granted microphone access (substitute** `timestamp` **or** `last_modified` **as appropriate):**  
`sqlite3 ~"/Library/Application Support/com.apple.TCC/TCC.db" "SELECT client, auth_value, last_modified FROM access WHERE service='kTCCServiceMicrophone' AND auth_value=2;"`

**View applications granted camera access (substitute** `timestamp` **or** `last_modified` **as appropriate):**  
`sqlite3 ~"/Library/Application Support/com.apple.TCC/TCC.db" "SELECT client, auth_value, last_modified FROM access WHERE service='kTCCServiceCamera' AND auth_value=2;"`

**Check which applications have screen recording permissions (substitute** `timestamp` **or** `last_modified` **as appropriate):**  
`sqlite3 ~"/Library/Application Support/com.apple.TCC/TCC.db" "SELECT client, auth_value, last_modified FROM access WHERE service='kTCCServiceScreenCapture' AND auth_value=2;"`

---

## XProtect

Learn how Apple’s built-in anti-malware solution, XProtect, silently helps protect macOS against malicious software by quarantining or blocking suspicious files. We’ll explore how its continuously updated signatures and behind-the-scenes security updates affect investigations, including how to monitor XProtect events through logs and quarantine attributes. We’ll also compare XProtect to other security measures like Gatekeeper and Notarization, discuss its minimal impact on forensic imaging, and highlight the newer “Remediator” component. Finally, we’ll cover best practices.

---

**Below are some commonly used commands for investigating XProtect activity and quarantine attributes on macOS. Note that for the** `--last <time>` **flag in** `log show`**, valid suffixes include s (seconds), m (minutes), h (hours), and d (days). There’s no direct suffix for weeks or months (e.g., use** `--last 7d` **for a week or** `--last 30d` **to approximate a month).**

- `log show --info --predicate 'process == "XProtectService"' --last 24h`  
    (Views recent XProtect-related log entries; adjust the time frame with 1s, 2m, 3h, 4d, etc.)
    
- `log show --info --predicate 'process == "XProtectRemediator"' --last 24h`  
    (Shows logs related to the XProtect Remediator component on modern macOS)
    
- `xattr -p com.apple.quarantine /path/to/file`  
    (Checks the quarantine attribute for a specific file)
    
- `ls -l@ /path/to/file`  
    (Lists extended attributes, which can help reveal quarantine info)
    

---

**Resources:**  
[The Secrets of XProtectRemediator](https://alden.io/posts/secrets-of-xprotect/)

---

## FileVault

Learn how FileVault 2 (FV2) provides data-at-rest protection on macOS through Full Disk Encryption (FDE). We’ll cover its evolution from home directory encryption to a more comprehensive solution, note its impact on forensic procedures (such as cold imaging), and outline simple best practices for investigators.

---

**To check FileVault status, use the following command:**  
`fdesetup status`

---

# macOS Logs

---

## Overview of the Unified Logging System

Learn about Apple’s move from older logging mechanisms, like the Apple System Log (ASL) and traditional UNIX-style syslog, to a centralized Unified Logging System introduced in macOS 10.12 (Sierra). You’ll see why logs once spread across multiple files now flow into a single compressed store and how this rolling buffer necessitates timely data capture. By the end, you’ll understand why Unified Logs are key to modern macOS forensics and be ready for deeper explorations in the lessons that follow.

---

## Unified Logs – System and Kernel Events

Learn how to search and filter system and kernel events in the macOS Unified Logging System using predicates, time-based queries, and real-time streaming. You’ll see how to spot suspicious kext loads, investigate launchd services, and uncover sandbox or AMFI code-signing checks—then preserve important data from the rolling log buffer by redirecting specific output to files for later analysis.

---

### Example System and Kernel Log Queries

Retrieve log entries whose process is set to "kernel," restricted to the past hour:  
`log show --predicate 'process == "kernel"' --last 1h`

Refine search to filter kext-related events in the kernel logs:  
`log show --predicate 'process == "kernel" AND eventMessage CONTAINS "kext"' --last 1h`

Stream kernel messages in real time (resource-intensive):  
`log stream --predicate 'process == "kernel"' --info`

Pull recent messages from the launchd subsystem (within the last 30 minutes):  
`log show --predicate 'subsystem == "com.apple.launchd"' --last 30m --info --style syslog`

Search for kernel events referencing sandbox or AMFI checks:  
`log show --predicate '(process == "kernel") AND (eventMessage CONTAINS "sandbox" OR eventMessage CONTAINS "AMFI")' --last 1h`

Redirect kernel logs to a file for later analysis:  
`log show --predicate 'process == "kernel"' --last 1h > kernel_events_last_hour.txt`

---

**Resources:**  
Here are updated log-specific resources with current links verified as of this writing:

**Apple’s Official Resources**

- [Apple Developer Documentation – Logging](https://developer.apple.com/documentation/os/logging)  
    Provides an overview of Apple’s Unified Logging System, including advanced features and usage examples.
    
- [Apple Support – macOS Logs and Console](https://support.apple.com/guide/console/welcome/mac)  
    Offers basic guidance on using the built-in Console app to view and filter macOS logs.
    

**GUI-Based Tools**

- Console (Built into macOS)  
    The native GUI for browsing Unified Logs. It can be slow with very large data sets but is good for smaller or quick investigations.
    
- [XProCheck, T2M2, Ulbow, and Consolation](https://eclecticlight.co/consolation-t2m2-and-log-utilities/)  
    Log utilities from The Eclectic Light Company.
    

**sysdiagnose (Built into macOS)**

- Captures extensive diagnostic data—including Unified Logs—for offline analysis. You can generate a sysdiagnose report by pressing **==Shift + Ctrl + Option + Command + Period==** simultaneously, or by running `sudo sysdiagnose` in Terminal.
    

These resources focus specifically on macOS logs, including how to handle `.tracev3` files—compressed archives created by the Unified Logging System—visualize large datasets, and supplement the command-line methods covered in this module.

---

## Unified Logs – Authentication and Security

Learn how macOS logs authentication attempts, from loginwindow entries tracking user logins or logouts to sudo/PAM events for privilege escalation. We'll also see how Gatekeeper (spctl) and TCC logs can be queried. Finally, learn how to preserve important data from the rolling log buffer by redirecting specific output to files for later analysis.

---

### Example Authentication and Security Log Queries

Retrieve log entries for the "loginwindow" subsystem (last hour):  
`log show --predicate 'subsystem == "com.apple.loginwindow"' --last 1h`

Refine the search to show only successful or failed login attempts (run each command separately as needed):  
`log show --predicate 'subsystem == "com.apple.loginwindow" AND eventMessage CONTAINS "success"' --last 1h`  
`log show --predicate 'subsystem == "com.apple.loginwindow" AND eventMessage CONTAINS "failed"' --last 1h`

An alternative approach is to look for “fail”, which would match “failed” and “failure”:  
`log show --predicate 'subsystem == "com.apple.loginwindow" AND eventMessage CONTAINS "fail"' --last 1h`

Locate entries related to sudo or PAM (privilege escalations):  
`log show --predicate '(process == "sudo") OR (eventMessage CONTAINS "PAM")' --last 1h`

Stream authentication events in real time (resource-intensive, live system only):  
`log stream --predicate 'subsystem == "com.apple.loginwindow" OR process == "sudo"' --info`

Pull Gatekeeper logs (syspolicyd) from the last 24 hours:  
`log show --predicate 'process == "syspolicyd"' --last 24h`

Pull Gatekeeper logs (spctl) from the last 24 hours (older versions of macOS):  
`log show --predicate 'process == "spctl"' --last 24h`

Inspect the security assessment subsystem (e.g., Gatekeeper decisions):  
`log show --predicate 'subsystem == "com.apple.security.assessment"' --last 24h`

Search for TCC "deny" events in the last hour:  
`log show --predicate 'eventMessage CONTAINS "TCC" AND eventMessage CONTAINS "deny"' --last 1h`

Redirect authentication-related logs to a file for correlation (last hour):  
`log show --predicate 'subsystem == "com.apple.loginwindow" OR process == "sudo"' --last 1h > auth_events_last_hour.txt`

---

## Unified Logs – Advanced Authentication and Security

As a continuation of our previous lesson, this lesson focuses on four additional processes that may record important authentication and security events: authd, opendirectoryd, sshd, and securityd. This will allow us to gain deeper insight into local and remote authentication events on macOS. We’ll also look at a demo of practical SSH log searches (including failed and accepted connections).

---

### Example Advanced Authentication and Security Log Queries

Retrieve log entries for authd from the last hour:  
`log show --predicate 'process == "authd"' --last 1h`

Specify a custom time window (example using January 1, 2025, from 10:00 to 11:00):  
`log show --predicate 'process == "authd"' --start '2025-01-01 10:00:00' --end '2025-01-01 11:00:00'`

Show log entries for opendirectoryd from the last hour:  
`log show --predicate 'subsystem == "com.apple.opendirectoryd"' --last 1h`

Show log entries for sshd from the last hour:  
`log show --predicate 'process == "sshd"' --last 1h`

Search for log entries containing “ssh” from the last hour:  
`log show --predicate 'eventMessage CONTAINS[c] "ssh"' --last 1h --info`

Show accepted/failed SSH logins from the last hour:  
`log show --predicate 'processImagePath CONTAINS[c] "sshd" && (eventMessage CONTAINS[c] "accepted" || eventMessage CONTAINS[c] "error")' --last 1h --info`

Pull log entries for securityd from the last hour:  
`log show --predicate 'process == "securityd"' --last 1h`

---

## Unified Logs – Firewalls and Proxies

Learn how macOS logs firewall and proxy activities by enabling the built-in Application Layer Firewall (ALF) and examining its high-level entries. See how to retrieve firewall logs in real time or via snapshot, and discover what’s captured when incoming connections are detected (including an nmap demo). Then explore how proxies are managed, with examples for searching proxy-related logs and pinpointing key events through timestamps and authentication data. Finally, learn to broaden your network analysis by checking general Unified Logs and considering deeper packet-level tools like pf, tcpdump, or Wireshark when ALF’s limited detail isn’t enough.

---

### Example Firewall and Proxy Log Queries

Check the status of the macOS firewall:  
`sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate`

View ALF events from the last 24 hours:  
`log show --predicate '(process == "socketfilterfw") OR (subsystem == "com.apple.alf")' --last 24h`

Monitor events in real time on a live system (resource-intensive):  
`log stream --predicate '(process == "socketfilterfw") OR (subsystem == "com.apple.alf")'`

Preserve firewall logs by saving them to a file:  
`log show --predicate '(process == "socketfilterfw") OR (subsystem == "com.apple.alf")' --last 24h > firewall_snapshot.txt`

Look for references to proxy configurations (last 24 hours):  
`log show --predicate 'eventMessage CONTAINS "proxy"' --last 24h`

Narrow the proxy search to a particular daemon or subsystem:  
`log show --predicate 'process == "configd" AND eventMessage CONTAINS "proxy"' --last 24h`

---

## Unified Logs – Wi-Fi and Network

Learn how to investigate macOS Wi-Fi and network activity, from identifying which networks a Mac joins to reviewing DHCP requests, interface changes, and local service discovery. You’ll also see how to check device continuity logs for features like Handoff or AirDrop, investigate VPN or advanced networking frameworks, and capture critical data before it rolls out of the Unified Log.

---

### Example Wi-Fi and Network Log Queries

View Wi-Fi logs for airportd (last 24 hours):  
`log show --predicate 'process == "airportd"' --last 24h`

Refine Wi-Fi logs to look for activity associated with a specific SSID:  
`log show --predicate 'process == "airportd" AND eventMessage CONTAINS "SSID-NAME-HERE"' --last 24h`

Monitor Wi-Fi events in real time (resource-intensive):  
`log stream --predicate 'process == "airportd"' --info`

Look for network configuration changes via configd:  
`log show --predicate 'process == "configd"' --last 24h`

Check broader network events on newer macOS releases:  
`log show --predicate 'subsystem == "com.apple.network"' --last 24h`

Look for local service discovery with mDNSResponder:  
`log show --predicate 'process == "mDNSResponder"' --last 24h`

Search rapportd logs for device continuity events:  
`log show --predicate 'process == "rapportd"' --last 24h`

For detailed logs covering AirDrop activity:  
`log show --predicate 'subsystem == "com.apple.sharing"' --last 24h`  
`log show --predicate 'process == "sharingd"' --last 24h`

Look for VPN or advanced network features (NetworkExtension subsystem):  
`log show --predicate 'subsystem == "com.apple.networkextension"' --last 24h`

Look for VPN or advanced network features (nehelper process):  
`log show --predicate 'process == "nehelper"' --last 24h`

Preserve a snapshot of key network events:  
`log show --predicate '(process == "airportd" OR process == "configd" OR process == "rapportd" OR subsystem == "com.apple.network")' --last 24h > network_events_snapshot.txt`


---

## Unified Logs – Bluetooth

Learn how to investigate macOS Bluetooth logs, from discovering nearby devices and pairing events to capturing Apple Watch unlock or continuity connections. You’ll see how to filter for specific daemons, monitor logs in real time, and save critical data before it’s overwritten—all helping you trace Bluetooth interactions, detect unusual pairings, and more.

---

### Example Bluetooth Log Queries

Show Bluetooth logs from bluetoothd or blued (last 24 hours):  
`log show --predicate 'process == "bluetoothd" OR process == "blued"' --last 24h`

Filter for pairing-related messages:  
`log show --predicate '(process == "bluetoothd" OR process == "blued") AND eventMessage CONTAINS "Pairing"' --last 24h`

Filter for connected/disconnected messages:  
`log show --predicate '(process == "bluetoothd" OR process == "blued") AND (eventMessage CONTAINS "Connected" OR eventMessage CONTAINS "Disconnected")' --last 24h`

Check wirelessproxd logs (Apple Watch unlock, proximity features):  
`log show --predicate 'process == "wirelessproxd"' --last 24h`

Search rapportd logs for continuity over Bluetooth:  
`log show --predicate 'process == "rapportd"' --last 24h`

Look at subsystem-level Bluetooth events:  
`log show --predicate 'subsystem == "com.apple.Bluetooth"' --last 24h`

Stream Bluetooth logs in real time (resource-intensive):  
`log stream --predicate '(process == "bluetoothd" OR process == "blued" OR process == "wirelessproxd")' --info`

Preserve a snapshot of Bluetooth events:  
`log show --predicate '(process == "bluetoothd" OR process == "blued" OR process == "wirelessproxd" OR subsystem == "com.apple.Bluetooth")' --last 24h > bluetooth_events_snapshot.txt`

---

## Unified Logs – Gatekeeper, TCC, and XProtect

Learn how to investigate macOS security events for Gatekeeper, TCC, and XProtect by focusing on syspolicyd (for code-signing checks and blocks), analyzing TCC permission grants or denials, and checking XProtect or its Remediator logs.

---

### Example Gatekeeper, TCC, and XProtect Log Queries

Gatekeeper (syspolicyd) logs (last 24 hours):  
`log show --predicate 'process == "syspolicyd"' --last 24h`

Older references (spctl) for Gatekeeper (rare on modern macOS):  
`log show --predicate 'process == "spctl"' --last 24h`

Basic TCC events (last 24 hours):  
`log show --predicate 'eventMessage CONTAINS "TCC"' --last 24h`

Filter for TCC denials:  
`log show --predicate 'eventMessage CONTAINS "TCC" AND eventMessage CONTAINS "deny"' --last 24h`

Check for TCC “grant” messages:  
`log show --predicate 'eventMessage CONTAINS "TCC" AND eventMessage CONTAINS "grant"' --last 24h`

Search for XProtect references (last 24 hours):  
`log show --predicate 'eventMessage CONTAINS "XProtect"' --last 24h`

Check the XprotectFramework subsystem (legacy references):  
`log show --predicate 'subsystem == "com.apple.XprotectFramework"' --last 24h`

Look for new XProtect Remediator logs:  
`log show --predicate 'eventMessage CONTAINS "xprotectservice" OR eventMessage CONTAINS "XProtectService"' --last 24h`

Monitor Gatekeeper, TCC, and XProtect in real time (resource-intensive):  
`log stream --predicate '(process == "syspolicyd") OR (eventMessage CONTAINS "TCC") OR (eventMessage CONTAINS "XProtect") OR (eventMessage CONTAINS "xprotectservice")' --info`

Preserve Gatekeeper, TCC, and XProtect logs (last 24 hours):  
`log show --predicate '(process == "syspolicyd") OR (eventMessage CONTAINS "TCC") OR (eventMessage CONTAINS "XProtect") OR (eventMessage CONTAINS "xprotectservice")' --last 24h > gatekeeper_tcc_xprotect_snapshot.txt`

---

## Unified Logs – Crash Reporting

Learn how to view macOS crash-related logs in the Unified Logs, focusing on key queries for unexpected terminations, spindump data, and kernel panics. Remember, crash logs aren’t just for buggy software—malware often crashes, leaving footprints in the Unified Logs. These logs may help you find suspicious processes, exploit attempts, or repeated crash patterns.

---

### Example Crash Reporting Log Queries

Show CrashReporter subsystem logs (last 24 hours):  
`log show --predicate 'subsystem == "com.apple.crashreporter"' --last 24h`

Search for “crash” or “exception” keywords:  
`log show --predicate 'eventMessage CONTAINS "crash" OR eventMessage CONTAINS "exception"' --last 24h`

Monitor crash-related events in real time (resource-intensive):  
`log stream --predicate 'subsystem == "com.apple.crashreporter" OR eventMessage CONTAINS "crash"' --info`

Check spindump references (hung processes/spins):  
`log show --predicate 'process == "spindump"' --last 24h`

Look for diagnosticd logs (coordinates crash/diagnostic data):  
`log show --predicate 'process == "diagnosticd"' --last 24h`

Search for kernel panics:  
`log show --predicate 'eventMessage CONTAINS "panic" OR subsystem == "com.apple.kext"' --last 24h`

Preserve crash-related logs:  
`log show --predicate 'subsystem == "com.apple.crashreporter" OR process == "spindump" OR process == "diagnosticd"' --last 24h > crash_logs_snapshot.txt`

---

## Legacy Logs

Learn how to investigate legacy logging on macOS, even though modern systems rely mostly on Unified Logs. Older log data or partial ASL entries can still offer valuable forensic insights—especially on upgraded systems or daemons that haven’t fully transitioned.

---

### Legacy Logs and Locations

system.log (Legacy main system log):  
`/var/log/system.log`

Compressed archives of system.log (historical data):  
`/var/log/system.log.*` (e.g., `system.log.0.gz`)

ASL or DiagnosticMessages (older ASL data):  
`/var/log/asl/` or `/var/log/DiagnosticMessages/`

Other log files:  
`/var/log/wifi.log`  
`/var/log/racoon.log`  
`/var/log/ppp.log`

ASL configuration file:  
`/etc/asl.conf`

View system.log:  
`less /var/log/system.log`

Search system.log:  
`grep "keyword" /var/log/system.log`

Check rotation archives:  
`ls -lh /var/log/system.log.*`

Examine wifi.log, ppp.log, or racoon.log similarly:  
`less /var/log/wifi.log`

Parse .asl files on older macOS versions:  
`syslog -r -f /var/log/asl/filename.asl`

Search system.log for suspicious terms:  
`grep -i "error" /var/log/system.log`

Check archived logs for historical data:  
`ls -lh /var/log/system.log.*`

Browse active logging:  
`tail -f /var/log/system.log`

Look for ASL databases:  
`ls -lh /var/log/asl/`

Preserve logs before rotation:  
`cp /var/log/system.log /safe_location/`  
`cp /var/log/system.log.0.gz /safe_location/`

---

## Application-specific Logs

Learn how to investigate application-specific logs on macOS, focusing on firewall tools like Little Snitch and LuLu and MDM and identity solutions like Jamf Pro and Jamf Connect. You’ll see how each app may store logs outside the Unified Logs or through dedicated subsystems—helping you identify suspicious network activity, policy deployments, or user decisions that might otherwise go unnoticed in a forensic investigation.

---

### Example Application-specific Log Queries and Locations

Little Snitch – View Logs:  
`littlesnitch log`

Little Snitch – View Real-Time Traffic Logs:  
`littlesnitch log-traffic`

LuLu – Stream Logs from the Unified Logs:  
`log stream --predicate="subsystem=='com.objective-see.lulu'" --debug`

LuLu – Older Versions or Artifacts:  
`/Library/Objective-See/LuLu`  
`/Library/Logs/LuLu`

Jamf Connect – Show Historical Logs (Last 30 Min):  
`log show --predicate 'subsystem == "com.jamf.connect"' --debug --last 30m > ~/Desktop/JamfConnect.log`

Jamf Connect – Stream Logs in Real Time:  
`log stream --predicate 'subsystem == "com.jamf.connect"' --debug`

Jamf Pro – Plain Text Logs:  
`/Library/Logs/Jamf/`  
`/usr/local/jamf/logs/`  
`/var/log/jamf.log` (primary log used by Jamf Pro)

Antivirus / Endpoint Security:  
`/Library/Logs/<VendorName>/`

VPN Clients (Non-Apple):  
`/Library/Logs/<Vendor>/`  
`~/Library/Logs/<Vendor>/`

Browser / Chat Apps:  
`~/Library/Application Support/<AppName>/`

System-Wide Logs Location:  
`/Library/Logs`

User-Specific Logs Location:  
`~/Library/Logs`

---

## Additional Topics and Tools

Learn about additional macOS logging topics, including Spotlight indexing, the auditd subsystem for low-level security events, and key miscellaneous logs or locations. We'll then explore three GUI-based log tools—Console, Ulbow, and Consolation—that offer convenient, powerful ways to browse the Unified Log. Throughout this lesson, you’ll see how these final data sources and utilities complement the built-in command-line approaches, ensuring no evidence is overlooked in your forensic analysis.

**Resources:**  
[macOS Log Cheat Sheet](https://cdn.13cubed.com/downloads/macos_log_cheat_sheet.pdf)  
[Ulbow and Consolation (The Eclectic Light Company)](https://eclecticlight.co/downloads/)

---

# macOS File Systems

---

## HFS+

Learn about the HFS+ file system, starting with its history and evolving role in macOS. We’ll cover its key on-disk structures and features, explore metadata including timestamps, demonstrate timestamp manipulation (timestomping), observe the behavior of extended attributes, and use The Sleuth Kit (TSK) to analyze an HFS+ disk image.

---

## APFS

Learn about APFS, Apple’s modern file system introduced in macOS High Sierra (10.13). We’ll discuss its Copy-on-Write (COW) design, container-based layout, native encryption, and snapshots, highlighting how these features differ from HFS+. You’ll also see timestamps in action and discover why traditional data acquisition workflows often fail on APFS volumes.

**Resources:**  
[Magnet AXIOM and macOS/APFS](https://www.magnetforensics.com/resources/white-paper-magnet-axiom-and-macos-apfs/)  
[Getting Saucy with APFS](https://www.mac4n6.com/blog/2018/5/26/presentation-slides-demo-videos-getting-saucy-with-apfs)

---

## exFAT

Learn about exFAT, introduced by Microsoft in 2006 as a successor to FAT32 and supported since Mac OS X 10.6.5 (Snow Leopard). We’ll explore how its simplicity and cross-platform focus differ from HFS+ and APFS, and where you'll likely encounter it in your investigations.

---

# macOS Core Forensic Artifacts

---

## Introduction

A quick overview of the key macOS forensic artifacts we'll explore in detail throughout the module: `.DS_Store`, the Trash, File System Events (FSEvents), `knowledgeC.db`, and Biome. Then we'll shift our focus to a very important macOS artifact parsing tool called mac_apt. We'll install mac_apt and run it against a disk image, not only solidifying our understanding of how that tool works, but also learning about additional supporting artifacts along the way.

---

## .DS_Store

Learn about `.DS_Store` files, the hidden macOS artifact that stores Finder view preferences. These files are very similar to `Desktop.ini` and ShellBags in Windows, and can be used to show which specific folders have been viewed in Finder (the macOS GUI). In this lesson, you’ll learn about the important information tracked in these files, their connection to trashed items—hinting at what’s coming next in our Trash lesson—and discover how to parse this important artifact.

**Resources:**  
[.DS_Store-parser](https://github.com/hanwenzhu/.DS_Store-parser) (Used in this lesson)  
[DSStoreParser](https://github.com/nicoleibrahim/DSStoreParser)  
[DSStoreParser](https://github.com/BeanBagKing/DSStoreParser) (Python 3 fork by Mike Peterson)  
[DSStoreView](https://github.com/macmade/DSStoreView)

---

## Trash

Learn how to investigate the macOS Trash by exploring user-specific `.Trash` folders on internal volumes and `.Trashes` folders on external volumes. You’ll see how "Put Back" references in `.DS_Store` reveal original file paths, discover how ctime can expose deletion timelines, and learn about other critical information hidden in this deceptively simple artifact.

**Resources:**  
[AppleSingle / AppleDouble: Python Parsing Library](https://formats.kaitai.io/apple_single_double/python.html)  
[AppleSingle / AppleDouble: Go Parsing Library](https://formats.kaitai.io/apple_single_double/go.html)

---

## File System Events (FSEvents)

Learn how to harness the power of File System Events (FSEvents) logs for effective timeline analysis and investigative insight. FSEvents is one of the most important macOS forensic artifacts available, capturing file creations, deletions, renames, modifications, and metadata changes—bridging the gaps left by normal file system metadata. If you have a Windows forensics background, you can think of FSEvents as being similar to the kind of information we can get from the USN Journal. Lastly, you'll see how these logs can be beneficial in a forensic scenario by walking through a practical demo of parsing FSEvents from a live system and interpreting the results.

**Resources:**  
[FSEvents: How They Work and Why They Matter for Mac Analysis](https://www.hexordia.com/blog/mac-forensics-analysis)

---

## knowledgeC.db

Learn about `knowledgeC.db` and the potentially valuable information it still holds. While many usage records once stored here have shifted to Apple’s newer Biome database, there may still be significant data in `knowledgeC.db`—especially on older systems. After covering the key details of this artifact, you’ll see how to analyze it using macOS’s built-in `sqlite3` command-line tool.

---

### Example knowledgeC.db SQLite Queries

Show all distinct event streams (provides an overview of the data types logged in `knowledgeC.db`):  
`SELECT DISTINCT ZSTREAMNAME`  
`FROM ZOBJECT`  
`ORDER BY ZSTREAMNAME;`

Show app usage (foreground apps and the start time of each usage event):  
`SELECT`  
`ZSTREAMNAME,`  
`ZVALUESTRING,`  
`DATETIME(ZSTARTDATE + 978307200, 'unixepoch') AS LocalStartTime`  
`FROM ZOBJECT`  
`WHERE ZSTREAMNAME LIKE '%app%'`  
`ORDER BY ZSTARTDATE DESC`  
`LIMIT 10;`

Show Bluetooth connection changes (connections/disconnections):  
`SELECT`  
`ZSTREAMNAME,`  
`ZVALUEINTEGER,`  
`DATETIME(ZSTARTDATE + 978307200, 'unixepoch') AS LocalStartTime`  
`FROM ZOBJECT`  
`WHERE ZSTREAMNAME = '/bluetooth/isConnected'`  
`ORDER BY ZSTARTDATE DESC`  
`LIMIT 10;`

Show web usage (web-related activity or Safari usage):  
`SELECT`  
`ZSTREAMNAME,`  
`ZVALUESTRING,`  
`DATETIME(ZSTARTDATE + 978307200, 'unixepoch') AS LocalStartTime`  
`FROM ZOBJECT`  
`WHERE ZSTREAMNAME = '/app/webUsage'`  
`ORDER BY ZSTARTDATE DESC`  
`LIMIT 10;`

Show media playback events (`nowPlaying`):  
`SELECT`  
`ZSTREAMNAME,`  
`ZVALUESTRING,`  
`DATETIME(ZSTARTDATE + 978307200, 'unixepoch') AS LocalStartTime`  
`FROM ZOBJECT`  
`WHERE ZSTREAMNAME = '/media/nowPlaying'`  
`ORDER BY ZSTARTDATE DESC`  
`LIMIT 10;`

---

## Biome

Learn how Biome has taken over many usage logs once stored in `knowledgeC.db` and `PowerLog`. You’ll also see why `knowledgeC.db` remains important on older systems, but why Biome is rapidly becoming a must-check artifact for modern macOS forensics. Unfortunately, open-source parsing solutions are still lagging, making commercial tools or waiting for official support in mac_apt your best option for analyzing Biome’s proprietary `SEGB` files.

**Resources:**  
[Python Modules for Parsing SEGB v1 and v2](https://github.com/cclgroupltd/ccl-segb)

---

## mac_apt + Additional Artifacts

Learn how mac_apt expands our macOS forensics toolkit by parsing a broad array of additional artifacts beyond the core set we’ve already studied. We’ll install mac_apt using a special script, run it against a sample disk image, and discover how it retrieves evidence from plist files, SQLite databases, and more. Along the way, we’ll analyze user activity revealed by plugins like RecentItems, Safari, Spotlight, and many others. By the end, we’ll see why mac_apt is a powerful solution for sifting through countless files and logs to uncover critical evidence, and how we can supplement its findings with manual checks and other forensic tools for a comprehensive investigation.

**ℹ️ About mac_apt:**  
mac_apt continues to evolve and is under active development. As a result, what you see on screen in the video may differ slightly from what you encounter in practice. The video reflects a snapshot in time and focuses on the core functionality you need to understand; any features and capabilities introduced later build upon this foundation. Always reference the details below, which will be updated to reflect any significant changes in installation or usage.

---

### mac_apt Installation (Installer Script)

- Download the [installer script](https://github.com/13Cubed/install_mac_apt/blob/main/mac_apt_Install_macOS.sh).  
    We have customized the installation script to correct an issue that can occur with specific versions of macOS. We recommend using this version instead of the one hosted in the official repo.
    
- Make the script executable:  
    `chmod 755 mac_apt_Install_macOS.sh`
    
- Run the installer script:  
    `./mac_apt_Install_macOS.sh`
    
- Activate the Python virtual environment:  
    `cd /path/to/mac_apt`  
    `source env/bin/activate`
    
- Verify the installation:  
    `python3 mac_apt.py`  
    You should see usage information and an error about missing required arguments.
    

### **🆕 arm64 Release (Alternate Option)**

- Download the latest [arm64 release](https://github.com/ydkhatri/mac_apt/releases/latest) and decompress it.
    
- Open Terminal (in Applications > Utilities > Terminal).
    
- Remove the quarantine attribute:  
    `xattr -dr com.apple.quarantine /path/to/mac_apt_arm64.app`  
    Replace `/path/to/` with the actual location of the app (you can drag the app into Terminal to fill the path automatically).
    
- Enter the directory:  
    `cd /path/to/mac_apt_arm64.app/Contents/MacOS`
    
- Verify the installation:  
    `./mac_apt`  
    You should see usage information and an error about missing required arguments.
    

---

### mac_apt Usage

Depending on the installation method:

**Installer Script:**

- Activate the Python virtual environment if it isn't already:  
    `cd /path/to/mac_apt`  
    `source env/bin/activate`
    
- Run mac_apt:  
    `python3 mac_apt.py -o /path/to/output -x SPARSE /path/to/sparseimage ALL`
    

**arm64 Release:**

- Run mac_apt:  
    `./mac_apt -o /path/to/output -x SPARSE /path/to/sparseimage ALL`
    

**Options:**

- `-o` specifies the output path, which will be created if it doesn't already exist.
    
- `-x` tells mac_apt to also save the output as an Excel spreadsheet. Without it, only a SQLite database is created; with it, both the database and spreadsheet are created.
    
- `SPARSE` tells mac_apt that the input type is a sparse image. Other options include `AFF4`, `AXIOMZIP`, `DD`, `DMG`, `E01`, `MOUNTED`, `VMDK` or `VR`.
    

Note that mac_apt only supports _uncompressed_ DMG images, which excludes those created by Fuji (a forensic acquisition tool we’ll cover in the upcoming Evidence Collection module). That’s why we’re using a sparse image here. To follow along, replace `/path/to/sparseimage` in the usage example with the path to the **acquisition1 image**, which you can download below. Alternatively, you can also download the **pre-created mac_apt output**, though we highly encourage you to run the tool yourself.

Using `ALL` runs every mac_apt plugin, the list of which can be viewed with `-h`. Using `FAST`, as the name implies, processes artifacts more quickly but omits iPhone/iPad backup databases, Spotlight indexes, and the Unified Log export.

Alternatively, a list of specific plugins can be specified, separated by a space. You can also disable certain plugins from `FAST` or `ALL` by appending `-` to the name: `ALL IDEVICEBACKUPS- WIFI-` will run ALL plugins except `IDEVICEBACKUPS` and `WIFI`.

---

**📦 Downloads:**  
[acquisition1_sparseimage.zip](https://files.13cubed.com/ime/acquisition1_sparseimage.zip)  
[acquisition1_mac_apt.zip](https://files.13cubed.com/ime/acquisition1_mac_apt.zip)

**Resources:**  
[macOS Core Forensic Artifacts Cheat Sheet](https://cdn.13cubed.com/downloads/macos_core_forensic_artifacts_cheat_sheet.pdf)  
[mac_apt Documentation](https://github.com/ydkhatri/mac_apt/wiki/Sample-Usage)  
[DB Browser for SQLite](https://sqlitebrowser.org/dl/)

---

### Sample Investigative Questions

- What is the name of the machine?
    
- Who is the primary user associated with the machine?
    
- When was the user's password last set, and what is their password hint?
    
- What timezone is currently configured for the system?
    
- When was the APFS data container created?
    
- What is the local IP address of the machine, and what SSID is it connected to?
    
- What games are installed, and what other non-work-related apps are present?
    
- What LLM tool is installed?
    
- Where is the LibreOffice document created by the user (file name and full path)?
    
- What email provider is accessed by the user?
    
- What 13Cubed app is set to open at the next login?
    
- What background task in the user's Downloads folder is set to run at login?

---

# Persistence Mechanisms

---

## Launch Daemons and Launch Agents

Learn how to identify and investigate macOS Launch Daemons and Launch Agents. Discover their roles as core background services and explore the differences between system-level daemons and user-level agents. We’ll discuss their locations, examine how XML Property List files control what and when they run, and see a Launch Daemon in action with a hands-on demo. Finally, we’ll review hunting guidance for detecting this common persistence mechanism.

**📦 Downloads:**  
[go4launch.dmg](https://cdn.13cubed.com/downloads/go4launch.dmg)

**Resources:**  
The following is a **sample XML Property List** that can be used for either a Launch Daemon or a Launch Agent, depending on where you place it.

Launch Daemon:  
Place it in `/Library/LaunchDaemons/`, where it typically runs with system privileges and no direct user interaction.

Launch Agent:  
Place it in `~/Library/LaunchAgents/` or `/Library/LaunchAgents/`, where it typically runs in a user session and can present a user interface.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.example.persistence</string>

  <key>KeepAlive</key>
  <false/>

  <key>ProgramArguments</key>
  <array>
    <string>/path/to/binary</string>
    <string>run</string>
  </array>

  <key>RunAtLoad</key>
  <true/>

  <!-- 
    Optional (often used in Launch Agents):
    <key>LimitLoadToSessionType</key>
    <string>Aqua</string>
  -->
</dict>
</plist>

```

---

## Privileged Helper Tools

Learn how to identify and investigate Privileged Helper Tools, which allow legitimate applications to perform privileged actions without constantly prompting for credentials. In this lesson, we’ll explore how attackers can exploit these tools, the steps to verify their code signatures, and the key indicators—such as suspicious names or tampered property lists—that may point to malicious activity. By the end, you’ll be equipped to spot, analyze, and mitigate compromised helper tools.

---

## Cron Jobs

Learn how to identify and investigate Cron Jobs on macOS. In this lesson, you’ll see why `cron`—despite Apple’s move to `launchd`—still operates on modern systems and can be exploited by attackers. We’ll discuss how to read a `crontab` entry, remove malicious jobs, and investigate related logs. By the end, you’ll know how to audit both user-level and system-wide `cron` configurations to uncover stealthy persistence threats.

---

## Login Items

Learn how attackers use Login Items to achieve user-level persistence on macOS. In this lesson, we’ll explore how these items can silently launch malicious scripts at user login, discuss deprecated hooks that can remain on older or upgraded systems, and cover the key red flags that indicate potential compromise. By the end, you’ll understand how to spot, remove, and further investigate user-level Login Items to uncover stealthy persistence mechanisms.

---

## System Extensions

Learn how to identify and investigate System Extensions in modern macOS environments. In this lesson, you’ll discover how Apple replaced kernel extensions (kexts) with user-space alternatives, how attackers still exploit them for persistence, and the practical steps to detect, verify, and remove malicious extensions that blend in with legitimate software.

---

## SSH Keys

Learn how SSH keys can be leveraged for persistence on macOS (and other UNIX and UNIX-like systems) by adding a public key to the `authorized_keys` file. In this lesson, we’ll demonstrate creating a key pair, placing the public key in a target user’s `authorized_keys` file, and logging into the system without a password. You’ll see how attackers use this method to maintain stealthy, passwordless access and why it’s crucial to review SSH configurations and file system timestamps for signs of malicious activity.

---

# Evidence Collection

---

## Unified Logs

Learn how to collect macOS Unified Logs for forensic analysis in this concise lesson. You’ll discover how to leverage the `log collect` command to gather a `.logarchive` in the tracev3 format – a richer method for preserving critical event data and metadata. We’ll cover targeting specific time windows, viewing logs offline without disturbing the live system, and ensuring a proper chain of custody. By the end, you’ll be able to extract detailed evidence from the macOS Unified Logging system and filter it as needed for your investigations.

---

## Fuji

Learn how to perform live, logical acquisitions on a Mac with Fuji, a free, open-source tool that captures existing files and generates compressed DMG disk images. We'll briefly introduce Fuji's advanced features like the Fuji Cartridge for recovery mode acquisitions. Then, we’ll explore the different acquisition methods. Finally, we’ll walk through a complete live acquisition workflow so you'll be ready to incorporate this powerful tool into your macOS forensic investigations.

---

### Running Fuji: Gatekeeper

Fuji is neither code signed nor notarized (two distinct things: code signing confirms the developer's identity via an Apple Developer certificate; notarization means Apple has also scanned it for malware). Because of this, Gatekeeper will block it from running by default. You have two options:

- **System Settings method:** Go to System Settings > Privacy & Security and click "Open Anyway" after the block prompt.
    
- **xattr method (recommended):** Strip the quarantine attribute macOS attaches to downloaded files before copying Fuji to your acquisition volume: `xattr -d com.apple.quarantine /path/to/Fuji.dmg`
    

This is common with open-source forensic tools. Fuji is well-known and trusted, but always verify the download hash before removing the quarantine attribute and running any tool.

**Note:** In the lesson video, the xattr method was already applied to the Fuji DMG prior to recording, so you won't see this step performed on screen.

---

### Converting DMG to Sparseimage

Fuji produces compressed DMG disk images. Some forensic tools, including mac_apt, may not work with the compressed DMG format. If you need to convert your Fuji acquisition to an uncompressed sparseimage for compatibility with such tools, you can easily do so using the macOS built-in `hdiutil` command.

To convert a compressed DMG to a sparseimage:

1. Open Terminal
    
2. Run the following command (replace paths with your actual file locations):  
    `hdiutil convert /path/to/image.dmg -format UDSP -o /path/to/image.sparseimage`
    
3. The conversion may take several minutes depending on the size of the image
    

**Note:** For the exercises in this course, sparseimage files are already provided where needed, so you won't need to perform this conversion for course materials.

---

**Resources:**  
[Fuji: Forensic Unattended Juicy Imaging](https://github.com/Lazza/Fuji/releases/latest)  
[Fuji Documentation](https://fujiapp.top/docs/overview/)

---

## Unix-like Artifacts Collector (UAC)

Learn how to use Unix-like Artifacts Collector (UAC), a live response collection script for Incident Response that makes use of native binaries and tools to automate artifact collection. You’ll see how UAC preserves the order of volatility, offers YAML-based artifact definitions and profiles for customized data collection, and creates comprehensive output archives for review and interpretation.

**ℹ️ About Unix-like Artifacts Collector (UAC):**  
UAC continues to evolve and is under active development. As a result, what you see on screen in the video may differ slightly from what you encounter in practice. The video reflects a snapshot in time and focuses on the core functionality you need to understand; any features and capabilities introduced later build upon this foundation. Always reference the details below, which will be updated to reflect any significant changes in installation or usage.

**Example of performing a triage collection with UAC:**  
`sudo ./uac -p ir_triage /path/to/output`

**📦 Downloads:**  
[macos-vm_uac.zip](https://files.13cubed.com/ime/macos-vm_uac.zip)

**Resources:**  
[Unix-like Artifacts Collector (UAC)](https://github.com/tclahr/uac)  
[Unix-like Artifacts Collector (UAC) Documentation](https://tclahr.github.io/uac-docs/)

---

## Acquiring Memory

Learn how to approach live memory acquisition on macOS. You’ll see why paid tools like Volexity Surge may be the only reliable choice. We’ll also talk about limitations associated with Volatility 3, and why Volexity Volcano might be your best bet for memory analysis.

**Resources:**  
[Volexity Surge](https://www.volexity.com/products-overview/surge/)  
[Volexity Volcano](https://www.volexity.com/products-overview/volcano/)  
[Volatility 3 macOS Tutorial](https://volatility3.readthedocs.io/en/latest/getting-started-mac-tutorial.html)  
[How to Perform Memory Forensic Analysis in macOS Using Volatility 3](https://cpuu.hashnode.dev/how-to-perform-memory-forensic-analysis-in-macos-using-volatility-3)

---

# Timelining

---

## UAC + mactime

Learn how to turn a UAC-generated bodyfile into a clear, human-readable file system timeline using mactime from The Sleuth Kit (TSK). We’ll also walk through filtering the data with `grep` to create a customized timeline that includes only the dates or metadata you need. Finally, we’ll review the output to understand how to interpret each field and effectively analyze the results.

---

## Plaso/Log2Timeline

Master the art of creating "super timelines" with Plaso/Log2Timeline, a powerful tool for digital forensic analysis. This lesson will guide you through the process of integrating and analyzing diverse data sources into a comprehensive timeline, enhancing your ability to uncover critical insights in forensic investigations.

**ℹ️ About Plaso/Log2Timeline:**  
Plaso/Log2Timeline continues to evolve and is under active development. As a result, what you see on screen in the video may differ slightly from what you encounter in practice. The video reflects a snapshot in time and focuses on the core functionality you need to understand; any features and capabilities introduced later build upon this foundation. Always reference the details below, which will be updated to reflect any significant changes in installation or usage.

---

### Plaso/Log2Timeline Installation

- Download [Plaso/Log2Timeline](https://github.com/log2timeline/plaso/releases/latest) and extract it:  
    `tar zxvf plaso-20xxxxxx.tar.gz`  
    `cd plaso-20xxxxxx`
    
- Install required system build tools:  
    `brew install pkgconf autoconf automake libtool`
    
- Create a Python virtual environment:  
    `python3.13 -m venv ~/plaso_env`
    
- Activate the virtual environment:  
    `source ~/plaso_env/bin/activate`
    
- Upgrade pip and setuptools:  
    `pip install --upgrade pip setuptools`
    
- Install Plaso and its dependencies:  
    `python3 -m pip install .`
    
- Verify the installation:  
    `log2timeline`  
    You should see usage information and an error about missing required arguments.
    

As Needed:

- Deactivate the virtual environment:  
    `deactivate`
    
- Re-activate the virtual environment:  
    `source ~/plaso_env/bin/activate`
    

---

### "Kitchen Sink" Collection with psteal

`psteal --source image.dmg -w timeline.csv`

### Parsing a UAC Collection with Log2Timeline

`log2timeline --storage-file timeline.plaso uac-HOSTNAME.local-macos-xxxxxxxxxxxxxx.tar.gz`

### Targeted Collection with Log2Timeline

Forensic Artifacts Definitions:  
`log2timeline --storage-file timeline.plaso image.dmg --artifact-filters macos`

YAML-based Filter Files:  
`log2timeline --storage-file timeline.plaso image.dmg --file-filter logs.yaml`  
  
**Example custom logs.yaml filter file:**

description: Include common macOS log directories
type: include
path_separator: '/'
paths:
  - '/var/log/.+'
  - '/Library/Logs/.+'
  - '/private/var/log/.+'
---
description: Exclude Apache webserver logs
type: exclude
path_separator: '/'
paths:
  - '/var/log/apache2/.+'

**You’ll find precreated YAML filter files within** `/Users/<username>/plaso_env/lib/python3.xx/site-packages/artifacts/data` **(assuming the "plaso_env" environment is created using the installation steps above).**

### Advanced Filtering with psort

Time Slicing:  
`psort -o dynamic -w timeline-filtered.csv timeline.plaso --slice 2024-06-09T22:00:00+00:00`

- Default slice parameter is 5 minutes
    
- Can be changed with `--slice-size x`, where `x` is minutes
    
- Time must be in ISO 8601 format with timezone offset
    

Date Ranges:  
`psort -o dynamic -w timeline-filtered2.csv`  
`timeline.plaso "date > '2023-12-31 23:59:59' and date < '2024-04-01 00:00:00'"`

---

**📦 Downloads:**  
[acquisition1_dmg.zip](https://files.13cubed.com/ime/acquisition1_dmg.zip)  
[acquisition1_supertimeline.zip](https://files.13cubed.com/ime/acquisition1_supertimeline.zip)

**Resources:**  
[Plaso Documentation](https://plaso.readthedocs.io/en/latest/)  
[Digital Forensics Artifact Repository](https://github.com/ForensicArtifacts/artifacts)

---

# Analyzing a Compromised System

---

## The Scenario

An introduction to the capstone scenario, a comprehensive practical exercise that immerses you in in-depth disk forensics. You'll gain hands-on experience investigating a security incident modeled after real-world intrusions, applying the knowledge and skills you've acquired throughout the course.

**📦 Disk Images and Supporting Evidence:**  
[acquisition2_sparseimage.zip](https://files.13cubed.com/ime/acquisition2_sparseimage.zip)  
[acquisition2_dmg.zip](https://files.13cubed.com/ime/acquisition2_dmg.zip)  
[acquisition2_logarchive.zip](https://files.13cubed.com/ime/acquisition2_logarchive.zip)  
[acquisition2_uac.zip](https://files.13cubed.com/ime/acquisition2_uac.zip)

---

## Getting Started

This lesson provides guidance to help you begin your investigation.

---

### 🚀 Examine the UAC Collection

We recommend starting by reviewing the Unix-like Artifacts Collector (UAC) collection. From the scenario, we know that Captain DeSoto received an email containing a link to a malicious application with an embedded command-and-control mechanism.

Decompress the UAC collection:  
Double-click on `uac-Mac.mynet-macos-20250517051144.tar.gz`  
Or use `tar zxvf uac-Mac.mynet-macos-20250517051144.tar.gz`

Examine the contents of the `live_response` directory. Are there any clues pointing to the IP address the malware connected to, or open files that provide insight into its behavior?

---

### 🛠️ Run mac_apt (Sparse Image Only)

Change into the directory where you installed `mac_apt`:  
`cd /path/to/mac_apt`

Activate the Python virtual environment:  
`source env/bin/activate`

Run `mac_apt` against the sparse image:  
`python3 mac_apt.py -o /path/to/output -x SPARSE /path/to/PWF6RWLW2G_Acquisition.sparseimage ALL`

By analyzing the resulting CSV output, you can quickly glean a variety of useful information.

---

### 📁 Query the Unified Logs Archive

In this scenario, there is also a separate Unified Logs collection that can be queried.

Query the log archive:  
`log show --archive /path/to/logs.logarchive --predicate [PREDICATE] [OPTIONS]`

---

### 💾 Mount the Disk Image (DMG or Sparse Image)

Based on information found in the UAC collection, `mac_apt` output, and Unified Logs, you will want to review the disk image directly to inspect the file system.

Mount the disk image (read-only):  
`hdiutil attach -readonly -noverify /path/to/PWF6RWLW2G_Acquisition.[sparseimage|dmg]`

The mounted volume will appear under `/Volumes` and will typically be named `Macintosh HD 1` if the current system volume is `Macintosh HD`.

---

### ✉️ Review Apple Mail Data

Since the scenario indicates that Captain DeSoto received an email leading to the initial compromise, it's important to review any email messages stored within the disk image.

Relevant messages can be found in Robert DeSoto’s user directory at the following path:  
`/Users/rdesoto/Library/Mail/V10/56523BFD-3ACA-42B4-92EC-2DB1555DE968/[Gmail].mbox`

---

### ⏳ Create a Super Timeline with Plaso/Log2Timeline (DMG Image Only)

Consider creating a super timeline using Plaso/Log2Timeline with the DMG version of the disk image.

Change into the directory where you installed Plaso/Log2Timeline:  
`cd /path/to/plaso`

Activate the Python virtual environment:  
`source ~/plaso_env/bin/activate`

Run `psteal` to create a comprehensive timeline:  
`psteal.py --source /path/to/PWF6RWLW2G_Acquisition.dmg -w /path/to/output`

---

As a reminder, these are suggested steps—you are free to deviate and use any of the forensic techniques covered throughout the course. Your objective is to build an investigative timeline that reconstructs the key events surrounding this compromise.

---

### Sample Investigative Questions

- What is the name of the machine?
    
- Who is the primary user associated with the machine?
    
- When was the user's password last set, and what is their password hint?
    
- What timezone is currently configured for the system?
    
- What is the local IP address of the machine, and what SSID is it connected to?
    
- What is the name of the USB drive inserted to transfer applications onto the device?
    
- From which email address is the link to the malicious macOS app sent to the Captain?
    
- What is the name of the additional useful file sent as an attachment to the Captain via email?
    
- How many files intended to assist the Captain with his poker game are present on the system?
    
- How are these poker-related files transferred to the machine for execution?
    
- What malicious applications are installed by adding them to the Applications folder?
    
- What commands are executed to stage the Captain's data for future exfiltration?
    
- What is the IP address of the C2 server the computer is connected to during data acquisition?
    
- What persistence mechanisms are installed to maintain connectivity to the C2?
    
- What anti-forensic techniques are employed by the threat actor to cover their tracks?
    

🍀 Good luck! Remember, you can check your work in the “**Incident Postmortem**” lesson.

---

## Incident Postmortem


**🛑 STOP: ONLY READ BELOW AFTER YOU'VE COMPLETED YOUR INVESTIGATION 🛑**

This final lesson presents a complete incident postmortem that outlines what actually happened during the compromise. It includes supporting evidence, key indicators of compromise, and a detailed timeline of activity. Use this to compare your own findings against the confirmed facts, identify any missed details, and strengthen your investigative process through validation and reflection.

---

### Evidence Collection

All evidence collected on 2025-05-17

| Evidence Item                        | Description              | Notes                                      |
|--------------------------------------|--------------------------|--------------------------------------------|
| PWF6RWLW2G_Acquisition.sparseimage   | 41.97 GB on disk         |                                            |
| PWF6RWLW2G_Acquisition.dmg           | 31.65 GB on disk         |                                            |
| UAC_collect_1                        | .tar.gz, 742.4 MB        |                                            |
| UAC_collect_2                        | .tar.gz, 749.2 MB        | Second attempt after C2 uncertainty        |
| logs_collect_1                       |                          |                                            |
| logs_collect_2                       |                          | Second attempt after C2 uncertainty        |

---

### IOCs

| Category       | IOC                                | Description                      | Type    | Notes                            |
|----------------|------------------------------------|----------------------------------|---------|----------------------------------|
| Victim         | Captain Robert DeSoto              | Name of victim                   | Host    |                                  |
| Victim         | rdesoto                            | Username                         | Host    |                                  |
| Victim         | WeDoNotInterfere1!                 | Captured password                | Host    |                                  |
| Victim         | NCC-42296                          | Hostname                         | Host    |                                  |
| Victim         | PWF6RWLW2G                         | Serial number                    | Host    |                                  |
| Victim         | robert.desoto@email.com            | Email address                    | Network |                                  |
| Victim         | desotorobert787@gmail.com          | Email address                    | Network |                                  |
| Adversary      | bvyxx55d@protonmail.com            | Email address                    | Network |                                  |
| Infra          | 134.122.51.228:3693                | C2 delivery channel              | Network |                                  |
| Infra          | 134.122.51.228:8080                | C2 communication channel         | Network |                                  |
| Infra          | 134.122.51.228:3333                | C2 communication channel         | Network |                                  |
| Infra          | 134.122.51.228:4444                | C2 communication channel         | Network |                                  |
| Infra          | 134.122.51.228:5555                | C2 communication channel         | Network |                                  |
| Capability     | ADV.zip                            | Archive from C2                  | Host    | SHA256: 33ddc1...be06f           |
| Capability     | MacOS.py                           | Meterpreter payload              | Host    | SHA256: 9529df...8052            |
| Capability     | Astro Data Visualizer.app          | Installed application            | Host    | Full Disk Access, login enabled  |
| Capability     | PokerTips_1.zip                    | Email attachment                 | Host    | Password: hood                   |
| Capability     | PokerTips_1                        | Poseidon variant                 | Host    | Detected, not removed            |
| Capability     | PokerTips_2                        | Poseidon variant                 | Host    | Moved to Trash                   |
| Capability     | PokerTips_3                        | Poseidon variant                 | Host    | Detected, not removed            |
| Capability     | install_setup_v1.2.0.dmg           | Uploaded payload (Banshee)       | Host    | Renamed to PokerTips_2           |
| Capability     | Launch.dmg                         | Payload from USB                 | Host    | Executed, not detected           |
| Capability     | LauncherForPokerTips1.dmg          | Payload from USB                 | Host    | Detected                         |
| Capability     | Predicting Bitcoin.app             | Payload from USB                 | Host    | Detected                         |
| Persistence    | com.system.update                  | Launch Agent (port 4444)         | Host    | Installed via Meterpreter        |
| Persistence    | com.system.updates                 | Launch Agent (port 5555)         | Host    | Installed via Meterpreter        |
| Exfiltration   | Botanical Data 1.numbers           | Exfiltrated file                 | Host    | From Desktop                     |
| Exfiltration   | /Users/rdesoto/Documents/CSV.zip   | Staged and exfiltrated           | Host    | Deleted after a few days         |
| Exfiltration   | /Users/rdesoto/Documents/XML.zip   | Staged and exfiltrated           | Host    |                                  |
| Evasion        | rm .zsh_history                    | Deleted shell history            | Host    |                                  |

---

### Summary Timeline

**2025-04-21 to 2025-05-08:**

- Initial Mac mini setup (user creation, hostname, settings)
    
- Region toggled (UK → US → UK) to enable iCloud with a U.S. number
    
- Timezone set to PDT
    
- Gmail configured; iCloud created using that account
    
- Downloaded `CSV` and `XML` astrobiology datasets
    
- Subscribed to astrobiology mailing lists
    
- Viewed astrobiology-related and retirement-themed YouTube videos
    

**2025-05-10:**

- Installed Xcode Command Line Tools and Python 3
    

**2025-05-12:**

- Received phishing email from Erik Pressman with `ADV.zip`
    
- Bypassed Gatekeeper; opened app manually via System Settings
    
- Initial Meterpreter shell established
    
- Full Disk Access prompts appeared; popups when accessing `Documents`
    
- Replied to adversary via email
    
- Installed Launch Agent `com.system.update` (port `4444`)
    
- Ran:
    
    - `enum_osx`
        
    - `password_prompt_spoof` (prompt appeared, password captured)
        
    - `keylog_recorder` (install failed)
        
- Installed second Launch Agent `com.system.updates` (port `5555`)
    
- Several reboots during shell reconnections
    
- Exfiltrated `Botanical Data 1.numbers` from `Desktop`
    

**2025-05-13:**

- Shell reestablished on port `8080`
    
- Ran:
    
    - `safari_lastsession` (no plist found)
        
    - `screen` (screenshot captured)
        
    - `keylog_recorder` (still failed)
        
    - `autologin_password` (failed escalation)
        
- From shell: zipped files into `CSV.zip` and `XML.zip`
    
- Both archives exfiltrated via Meterpreter
    

**2025-05-14:**

- Granted Full Disk Access to `Astro Data Visualizer.app`
    
- Received new phishing email with `PokerTips_1.zip`
    
- Extracted three Poseidon variants: `PokerTips_1`, `PokerTips_2`, and `PokerTips_3`
    
- `PokerTips_2` detected and moved to Trash
    
- Other variants remained in the `Documents` folder
    
- Installed Rosetta to support App Store `Texas Hold 'Em` app
    
- Ran:
    
    - `hashdump` (failed)
        
    - `enum_messages` (successful; data exfiltrated)
        
- Uploaded `install_setup_v1.2.0.dmg` and renamed it to `PokerTips_2`
    

**2025-05-15:**

- Ran privilege escalation modules:
    
    - `root_no_password` (failed; shell written to `/tmp`)
        
    - `sudo_password_bypass` (target not vulnerable)
        
- Attempted to run `PokerTips_2` via shell and Meterpreter
    
- Ran various recon commands: `ifconfig`, `ipconfig`, `cat`
    

**2025-05-16:**

- USB device discovered with:
    
    - `Launch.dmg` (executed; password prompt; ran silently)
        
    - `LauncherForPokerTips1.dmg` (executed; flagged as malware)
        
    - `Predicting Bitcoin.app` (executed; flagged as malware)
        
- Deleted `CSV.zip`; left `XML.zip`
    
- Ran `rm .zsh_history` (note: shell sessions still persisted elsewhere)
    

**2025-05-17:**

- Final shell session active
    
- Ran `enum_osx`
    
- Completed forensic acquisition

---

