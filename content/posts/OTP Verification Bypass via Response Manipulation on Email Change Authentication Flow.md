---
title: "OTP Verification Bypass via Response Manipulation on Email Change Authentication Flow"
date: 2026-04-2
tags: ["Otp", "Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough."
---


``` Platform: HackerOne ```
``` Status: Duplicate ```
---
<b><h1>Overview</h1></b>

I was testing a target belonging to a large online fashion platform, focusing on how the application handled response manipulation within its OTP verification flow.

During my assessment, I analyzed the client-server interaction to determine whether the OTP validation logic was enforced strictly on the backend or could be influenced through modified responses.

OTP-based authentication flows are a common area of interest in my testing, as improper validation can lead to account takeover or unauthorized actions when not implemented securely.

<img width="668" height="576" alt="Fq1m2EmXwAAYWAI" src="https://github.com/user-attachments/assets/18c3705b-2668-4e97-be48-1cf1dcd97350" />

The application has a feature where you could authorize an email change. But to change the email, it requires an otp to be sent to the current email address. So this where the real attack begins.
<img width="1145" height="392" alt="Screenshot (120)" src="https://github.com/user-attachments/assets/9bc3de1a-7d05-43be-97b3-5f036c01092a" />

When I clicked on the <b>change</b> button to update the email address, the application sent an OTP to the current email linked to the user account.

<img width="675" height="447" alt="Screenshot (121)" src="https://github.com/user-attachments/assets/98cb377a-4edf-48fc-a769-9361e7c25100" />

I turned on interception in Burp Suite and entered an invalid OTP to capture the request and response flow.

<img width="1764" height="750" alt="Screenshot (122)" src="https://github.com/user-attachments/assets/42eefd9a-ea5a-4419-85c9-90a6e65baa59" />

Next, I enabled <b>Intercept Response</b> to observe and manipulate the server’s response.

<img width="1310" height="511" alt="Screenshot (123)" src="https://github.com/user-attachments/assets/965c717f-fac6-428a-845e-dd61833c5eb2" />

<img width="630" height="279" alt="Screenshot (124)" src="https://github.com/user-attachments/assets/8028ca50-a3d7-408c-a0d8-3dc5c30b82b0" />

At this point, I recalled a valid `200 OK` response I had seen earlier while testing another endpoint. That gave me an idea of what a successful response should look like.

While interception was still active, I copied the success message format and replaced the error message from the OTP response with it.

<img width="623" height="611" alt="Screenshot (125)" src="https://github.com/user-attachments/assets/be6a3b32-95e1-4e30-802c-f5bad234eed1" />

After replacing the error message with the success message, I forwarded the modified response.

<img width="815" height="643" alt="Screenshot (126)" src="https://github.com/user-attachments/assets/e2b6b26e-4039-4115-9617-a49670036217" />

The application accepted the modified response and treated the OTP as valid. This allowed me to proceed to the next step, where I could input a new email address under my control and request an OTP to it.

<img width="1298" height="411" alt="Screenshot (127)" src="https://github.com/user-attachments/assets/15aaf437-732b-45bc-8051-e94fbb042e7a" />

An OTP was successfully sent to the new email address.

<img width="1319" height="427" alt="Screenshot (128)" src="https://github.com/user-attachments/assets/0c0aec56-eb5c-46e0-ba3f-582d9435953a" />

I then used the OTP from the new email to complete the process and successfully changed the account’s email address.

<img width="1382" height="132" alt="Screenshot (129)" src="https://github.com/user-attachments/assets/83d3b93c-b64f-4e3d-b4d9-b4ab656dc417" />

This could lead to an account takeover if an attacker already has your password but no access to your email.

<h1>IMPACT</h1>
An attacker who knows the victim's password
can bypass the current email OTP verification
and proceed directly to the new email
confirmation step, removing one layer of
account protection.

## Realistic Attack Scenario
<li>Attacker has your password</li>
(from phishing or data breach)
<li>Normally blocked because they don't</li>
have access to your email inbox for OTP
<li>With this bypass they skip that step</li>
<li>They are only stopped by the second OTP
sent to the NEW email they control</li>



