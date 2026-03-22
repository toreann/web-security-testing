# API REPLAY PROTECTION ANALYSIS (BURP SUITE)

## OVERVIEW
This project documents a web application security test focused on replay attack resistande in a promotional API endpoint.

The objetive was to determine if previously captured request could be resent or modified to generate duplicate rewards.

## OBJECTIVES
- Intercept promotional API request.
- Replay the request to evaluate duplicate redemption protection.
- Observe server-side validation behavior.

## TOOLS
- Burp Suite Community Edition.
- Firefox

## METHODOLOGY

### 1- Intercepting the request
Burp Proxy was configured to intercept traffic between the browser and web application

1- Configure browser proxy
2- Enable Burp Proxy Interception
3- Trigger the promotional action on the website
4- Capture the API request

Example request structure and request body
```http:
POST /api/promo/redeem HTTP/2
Host: api.example.com
Content-Type: application/json
```
```JSON:
"userId": 12345,
"email": "user@example.com",
"phone": "+1XXXXXXXXXX",
"promoId": "PROMO2026"
```

### 2- Replay Testing

The intercepted request was sent to Burp Repeater
Steps: 
1- Send Captured Request to Repeater
2- Resend the exact same request without modification
3- Perform controlled variations of user-related fields to determine duplicate promotion bypassing control

- Example request body:
```JSON:
 	"firstName": "John",
 	"lastName": "Doe",
 	"email": "user@example.com",
 	"phone": "+1XXXXXXXXXX",
 	"promoId": "PROMO2026",
 	"uid": 12345
```
---
- Replay attempt in the exact captured request:

- The server returned a response indicating promotion had already been claimed:
```JSON:
 	"errorCode": "PromoDuplicated",
 	"message": "Promotion already redeemed"
```
```http response:
	HTTP/2 400 Bad Request
```
- This indicates the backend maintains server-side redemption state and blocks duplicate requests.

---
- Parameter variation testing:
- To evaluate duplicate protection, several request variations were tested, one at a time.

- Email Variation: "email": "user+1@example.com";
- Phone Number Variation: "phone": "+1XXXXXXXXXY";
- Name Variation: "firstName": "Jane";
- UID Manipulation: "uid": 12346;

- Result: All manipulations had a rejected request and returned the message "PromoDuplicated"


## SECURITY CONCEPTS DEMONSTRATED
	HTTP request interception
	Replay attack testing
	API request manipulation
	Business logic security validation
	Server-side enforcement of redemption rules

## RISK ASSESSMENT

### Findings
- Promotion API replay attempt blocked by server-side duplicate detection
- The promotional redemption endpoint was tested for replay attack behavior by resending an intercepted HTTP request through Burp Suite Repeater.
- The server responded with a duplicate redemption error and prevented additional reward issuance.
---
- Severity Low:
- The application correctly prevents basic replay attacks by validating redemption state on the server side.
---
- Impact:
```If this protection were absent, an attacker could potentially:

resend a captured request
generate multiple promotional rewards
abuse the promotion system
```
This could result in financial loss or promotion abuse.
---

### Recommendation:
- Maintain server-side validation to ensure each promotion can only be redeemed once per eligible user.
Additional defensive measures may include:

- redemption state stored server-side
- rate limiting on redemption endpoints
- promotion usage logging
- monitoring for abnormal request patterns

---
## Testing Summary

| Test                     | Result  |
|--------------------------|---------|
| Request Replay           | Blocked |
| Email Casing Variation   | Blocked |
| UID Manipulation         | Blocked |
| Duplicate Redemption     | Blocked |

# LESSONS
- This analysis highlights the importance of enforcing promotion eligibility checks on the server side rather than trusting client-supplied parameters.
- Even when request fields such as email, phone number, name, or UID are modified, the backend can maintain integrity by validating the request against trusted internal user data.

# DISCLAIMER
This repository presents a sanitized educational case study demonstrating API replay-protection analysis techniques.

All identifying information, domains, endpoints, and personal data have been removed or replaced with generic examples. The content is intended solely for educational and documentation purposes.
