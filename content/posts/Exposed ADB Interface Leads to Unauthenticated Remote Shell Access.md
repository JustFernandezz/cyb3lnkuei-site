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

