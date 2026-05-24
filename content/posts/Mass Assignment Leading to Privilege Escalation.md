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
The application is vulnerable to Mass Assignment, allowing client-supplied parameters to be automatically mapped to backend objects without proper restrictions on sensitive attributes.

During testing, a legitimate request was made to retrieve user details, which returned user information including the current role value <b>(Account)</b>. The application later accepted additional privilege-related parameters manually inserted into the update request, specifically <b>role</b> and <b>roleId</b>.

Because the backend failed to enforce server-side restrictions on modifiable attributes, the application processed the injected parameters and updated the user's role to <b>Admin</b>, resulting in unauthorized privilege escalation.

