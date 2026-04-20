---
title: "OTP Verification Bypass via Response Manipulation on Email Change Authentication Flow"
date: 2024-01-01
tags: ["Otp"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
---
<h3>Status: Duplicate</h3>

<h3>Severity: Medium</h3>

---
<b><h1>Overview</h1></b>

I was testing this target's application, a popular online fashion company, so I thought of testing how the application handled response manipulation in the OTP flow.
OTP bypass vulnerability is one of my favorite vulns to look for while testing.

<img width="668" height="576" alt="Fq1m2EmXwAAYWAI" src="https://github.com/user-attachments/assets/18c3705b-2668-4e97-be48-1cf1dcd97350" />

The application has a feature where you could authorize an email change. But to change the email, it requires an otp to be sent to the current email address. So this where the real attack begins.
<img width="1145" height="392" alt="Screenshot (120)" src="https://github.com/user-attachments/assets/9bc3de1a-7d05-43be-97b3-5f036c01092a" />

When you click on <b>c</b>
