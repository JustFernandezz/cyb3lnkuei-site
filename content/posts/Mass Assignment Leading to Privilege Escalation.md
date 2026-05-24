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

Platform: ```Financial Application```

## Description
The application is vulnerable to Mass Assignment, allowing client-supplied parameters to be automatically mapped to backend objects without proper restrictions on sensitive attributes.

During testing, a legitimate request was made to retrieve user details, which returned user information including the current role value <b>(Account)</b>. The application later accepted additional privilege-related parameters manually inserted into the update request, specifically <b>role</b> and <b>roleId</b>.

Because the backend failed to enforce server-side restrictions on modifiable attributes, the application processed the injected parameters and updated the user's role to <b>Admin</b>, resulting in unauthorized privilege escalation.

## Discovery
I was testing account management functionality and reviewing the profile update flow. The application allowed authenticated users to retrieve and update account details through an API endpoint.

The initial process looked straightforward:
Step 1 → Retrieve account information

Step 2 → Application returns user profile data

Step 3 → Modify profile fields

Step 4 → Save changes

Nothing unusual.

I captured the request and reviewed the server response. The response returned various account attributes including the user's current role:
```
{
  "firstName":"John",
  "email":"user@example.com",
  "role":"Account"
}
```
Image:
<img width="1242" height="742" alt="IMG_1310" src="https://github.com/user-attachments/assets/c0b3b4e7-6e90-46ce-ab0a-291353e29f8a" />

At first glance, this looked like normal profile information. But seeing role values returned by the API immediately raised a question:

If the backend returns role information, is it also trusting role-related fields during updates?

The application UI never exposed any role modification functionality, but APIs often reveal more than the frontend intends.

Also, I had an idea of different roles and roleIds that existed for an admin,account or supervisor account which I got from an admin test account and also from inspecting Javascript files of the applications. So an admin account was provisioned and also a regular user account. This test is being done on the regular user account to see if we can escalate privileges.

## Exploitation
I intercepted the profile update request using Burp Suite.

The original request looked like this:
```
{
   "firstName":"John",
   "lastName":"Doe",
   "email":"user@example.com",
   "phoneNumber":"080XXXXXXXX"
}
```

I modified the request and manually added two additional parameters:
```
{
   "firstName":"John",
   "lastName":"Doe",
   "email":"user@example.com",
   "phoneNumber":"080XXXXXXXX",
   "role":"Admin",
   "roleId":"ce758156-2ad0-4fbf-bc01-59ca8aeecb6d"
}

```
Image:
<img width="1242" height="726" alt="IMG_1312" src="https://github.com/user-attachments/assets/e1c15f80-461f-407f-b61c-ccb950813843" />

Forwarded the request.

The following screenshots show the exact request and server response captured during testing:

Initial response showed the account role as Account

Modified request injected role and roleId

Server accepted and processed both values

The response came back:
```
{
   "isSuccess": true,
   "message":"Staff updated successfully",
   "data":{
      "role":"Admin"
   }
}

```
Image: <img width="1242" height="847" alt="IMG_1313" src="https://github.com/user-attachments/assets/adaf3ae1-0854-4a61-ad53-cb8da1eb5d9b" />

We now have admin privileges. Look at the top right corner of the image below and you see it's now an admin account:
<img width="1241" height="716" alt="IMG_1311" src="https://github.com/user-attachments/assets/5d116c62-9da6-4c9f-a483-28dcc7e5e664" />

## Impact

Any authenticated user capable of modifying their profile request could potentially escalate privileges by supplying unauthorized attributes.

This could allow attackers to:

- Gain administrative access
  
- Bypass role-based restrictions
  
- Access sensitive functionality
  
- Manage other users
  
- Fully compromise the application environment

## Remediation
Implement server-side allowlisting of accepted parameters

Reject sensitive attributes such as:
- Role
- RoleId

Use DTOs instead of directly binding request objects

Enforce authorization checks for all privilege modifications

Restrict role assignment functionality to administrator-only backend operations
