---
title: "Exposed Unrestricted Google API Key in Client-Side: JavaScript Allows Unauthorized API Abuse"
date: 2026-03-04T00:00:00Z
tags: ["Key", "Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
draft: false
---


```Target: uk.romwe.com ```
``` Platform: HackerOne ```
``` Status: Duplicate ```
``` Severity: Low ```

## Background
When I started hunting on SHEIN's bug bounty program on HackerOne, I wasn't looking for anything specific. I just had Burp Suite running, DevTools open, and curiosity doing the heavy lifting.
My usual starting point when hitting a new target is the page source and global JavaScript objects. A lot of developers don't realize how much sensitive information gets baked into server-rendered config objects that ship to every single visitor's browser. SHEIN was no different.


## Discovery
I opened the console on uk.romwe.com and typed one line:
 ```console.log(gbCommonInfo.GOOGLE_API_KEY)``` 
 
Returned the following key:
```"AIzaSyANkdKFbzWu5bgALBRqmT6xpUxdl_ncBBY"```

## POC — Step 1: Key Found in gbCommonInfo
The key was visible directly in the browser DevTools console under the gbCommonInfo object alongside other configuration data including Facebook App ID, Google Client ID, and reCAPTCHA site key.
<img width="739" height="790" alt="Screenshot_(89)" src="https://github.com/user-attachments/assets/c2d526ad-7813-47a7-9aa8-5fe2adc39cbd" />

## POC — Step 2: Directions API Works from External Origin
Finding a key is one thing. Proving it actually works from outside their infrastructure is what makes it a finding worth reporting.
I opened my terminal and fired a simple curl request, no cookies, no session, no referrer header. Just the key and a test query:
```curl "https://maps.googleapis.com/maps/api/directions/json?origin=London&destination=Manchester&key=AIzaSyANkdKFbzWu5bgALBRqmT6xpUxdl_ncBBY"```
<img width="1920" height="1080" alt="Screenshot_(88)" src="https://github.com/user-attachments/assets/561b4dd2-0c9e-4e34-acf0-e007ceff1f61" />

That ```"status": "OK" ``` told me everything. The key had no domain restrictions, no IP restrictions, and no API restrictions. It was completely open.
I kept going:
# Directions API
```curl "https://maps.googleapis.com/maps/api/directions/json?origin=London&destination=Manchester&key=KEY"```
```# status: OK ```

# Places API  
```curl "https://maps.googleapis.com/maps/api/place/findplacefromtext/json?input=london&inputtype=textquery&key=KEY"```
```# status: OK```

# Elevation API
```curl "https://maps.googleapis.com/maps/api/elevation/json?locations=51.5,-0.1&key=KEY"```
```# status: OK```

## Impact
- Financial impact   — billable API calls charged to ROMWE's Google Cloud account
  
- Availability impact — quota exhaustion disables location features for real users
  
-  No restrictions    — no domain, IP, or API-level restrictions configured

Cost estimate per 1,000,000 calls:
  Geocoding API   →  $5,000
  Places API      →  $32,000
  Directions API  →  $10,000
  Elevation API   →  $5,000

## Remediation
- Immediately rotate and revoke the exposed key in Google Cloud Console
  
- Restrict the new key to specific HTTP referrers: *.romwe.com/* and *.shein.com/*
  
- Restrict to only the specific APIs the application actually uses
  
- Never embed API keys in client-side JavaScript — proxy Google API calls through a backend service
