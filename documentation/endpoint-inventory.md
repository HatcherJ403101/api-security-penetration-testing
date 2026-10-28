# API Endpoint Inventory

## 1. Assessment Overview

**Application:** OWASP crAPI  
**Environment:** Local Docker lab  
**Testing tool:** Burp Suite Community Edition  
**Stage:** Initial authenticated API reconnaissance

## 2. Objective

Identify API endpoints used during normal registration, authentication and dashboard access. Record their HTTP methods, observed responses, authentication requirements and apparent purpose before conducting security tests.

## 3. Observed Endpoints

### Endpoint 01 — Login

- **Method:** POST
- **Path:** `/identity/api/auth/login`
- **Authentication:** Credentials supplied in request body
- **Observed result:** Login successful; response contained an authentication token
- **Status code:** Not recorded
- **Purpose:** Authenticate the test user and obtain a token

**Security notes:** Credentials and authentication tokens are sensitive. They must not be included in public reports or screenshots.

### Endpoint 02 — Authentication Verification

- **Method:** POST
- **Path:** `/identity/api/auth/verify`
- **Authentication:** Token supplied to the verification operation; exact request format to be confirmed from the sanitised request
- **Observed status:** HTTP 200
- **Purpose:** Verify an authentication token, consistent with the documented crAPI API behaviour

**Security notes:** An HTTP 200 response does not independently establish that every aspect of the verification process is secure.

### Endpoint 03 — User Dashboard

- **Method:** GET
- **Path:** `/identity/api/v2/user/dashboard`
- **Authentication:** Authorization header present
- **Observed status:** HTTP 200
- **Purpose:** Retrieve dashboard data for the authenticated user

**Security notes:** Confirm which fields are returned and whether they are appropriate for the authenticated user. Do not publish personal data, tokens or other sensitive response values.

## 4. Initial Observations

- The login flow returned an authentication token.
- An authentication-verification request returned HTTP 200.
- A subsequent dashboard request included an Authorization header and returned HTTP 200.
- These observations establish a baseline for normal authenticated API activity.
- No vulnerability has been confirmed from these requests alone.

## 5. Next Investigation Steps

1. Record the relevant request and response metadata for each endpoint.
2. Review the dashboard response fields using the local test account.
3. Establish expected behaviour for authenticated and unauthenticated requests.
4. Assess authorisation controls using only the local crAPI environment and test accounts.
5. Document any confirmed findings with sanitised evidence, impact, remediation and retest criteria.

## 6. Evidence Handling

Do not publish passwords, Bearer tokens, session cookies, API keys or other authentication secrets. Any request or response screenshots must be sanitised before they are added to GitHub.
