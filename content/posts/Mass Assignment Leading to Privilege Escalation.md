---
title: "Mass Assignment Leading to Privilege Escalation"
date: 2026-05-04T00:00:00Z
tags: ["Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
draft: false
---
Severity: ```Critical```

Type: ```Penetration Test Assessment```

## Description
The staff profile update endpoint does not restrict which fields a user can modify. By injecting role and roleId into the update request body, a regular Account-level user can self-promote to Admin — gaining full administrative access without any server-side authorisation check.

