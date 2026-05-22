---
title: "Exposed ADB Interface Leads to Unauthenticated Remote Shell Access"
date: 2026-02-04T00:00:00Z
tags: ["Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
draft: false
---

Severity: ```Critical```

Type: ```Penetration Test Assessment```

Vulnerability Class: ```Misconfiguration / Exposed Debug Interface```

Target: ```Android Device on Internal Network (10.10.30.109)```

## Summary
An Android device on the internal network was discovered with its ADB debug interface exposed on port 5555/tcp with no authentication required. By connecting remotely via ADB, an attacker with network access gains an unauthenticated shell on the device, with the ability to read/write files, install applications, extract data, and potentially pivot further into the network.

Discovery
During an internal network scan, Nmap was run against the 10.10.30.0/24 subnet. The scan revealed two live hosts of interest:

<img width="1920" height="481" alt="Android debugger" src="https://github.com/user-attachments/assets/a170783b-fc8e-4d5e-b327-a76eced6587b" />

Port 5555 is the default ADB-over-TCP port. Its presence on a network-accessible interface immediately indicated a misconfigured debug build or a device with persistent ADB-over-network enabled.

## Exploitation
Step 1 — Connect via ADB

With network access to the target, connecting is trivial:
```adb connect 10.10.30.109:5555```

No PIN, no pairing code, no authentication prompt — connection accepted immediately.

## Step 2 — Spawn Remote Shell
``` adb shell ```
<img width="1920" height="280" alt="Android debugger (2)" src="https://github.com/user-attachments/assets/d2570dbb-3f28-4da9-b919-01c265c02535" />

A shell was obtained instantly. The whoami output confirms access as the shell user — a highly privileged account on Android with access to most of the filesystem and the ability to escalate to root on debug builds.

``` 
ohm:/ $ ls
acct        cache         data_mirror    etc
apex        config        debug_ramdisk  init
bin         d             default.prop   init.environ.rc
bugreports  data          dev            init.recovery.amlogic.rc
...
```
Full filesystem listing returned, including sensitive directories such as /data, /sdcard, and /metadata.

## Root Cause
ADB over TCP (adb tcpip 5555) was left enabled on a network-accessible interface, likely from a development or debugging session that was never disabled before deployment. The device name ohm and the init.recovery.amlogic.rc file suggest this is an Amlogic-based Android TV or IoT device running a debug-enabled firmware build.

## Remediation Recommendations

- Disable ADB over TCP on all production devices — adb usb or set persist.adb.tcp.port to disabled
  
- Use USB-only ADB for any legitimate debugging needs

- Enable ADB authentication — Android 4.4+ supports RSA key pairing; enforce it

- Network segmentation — IoT/Android devices should not be reachable from general internal network ranges
