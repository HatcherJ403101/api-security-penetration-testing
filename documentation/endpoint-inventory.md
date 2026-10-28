# API Endpoint Inventory

## 1. Assessment Overview

**Application:** OWASP crAPI  
**Environment:** Local Docker lab  
**Testing tool:** Burp Suite Community Edition  
**Stage:** Initial API reconnaissance and authentication testing

## 2. Objective

Identify API endpoints used during normal registration, authentication and dashboard access. Record their HTTP methods, observed responses, authentication requirements and apparent purpose before conducting further security tests.

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
- **Authentication:** Authorization header present in the baseline request
- **Observed baseline status:** HTTP 200
- **Purpose:** Retrieve dashboard data for the authenticated user

**Security notes:** Confirm which fields are returned and whether they are appropriate for the authenticated user. Do not publish personal data, tokens or other sensitive response values

### Endpoint 04 — Add Vehicle

- **Method:** POST
- **Path:** `/identity/api/v2/vehicle/add_vehicle`
- **Authentication:** Bearer token present
- **Observed status:** HTTP 200
- **Observed result:** `Vehicle saved successfully`
- **Purpose:** Register a vehicle to the authenticated test account using a VIN and PIN.

**Security notes:** The request contains vehicle-registration information and an authentication token. Do not include the VIN, PIN, token or other sensitive values in public screenshots or reports.

**Assessment status:** Normal behaviour confirmed. Access-control testing remains to be performed.

### Endpoint 05 — Vehicle Location

- **Method:** GET
- **Path:** `/identity/api/v2/vehicle/<vehicle-id>/location`
- **Authentication:** Bearer token present
- **Observed status:** Record the actual status code from Burp Suite
- **Purpose:** Retrieve location information for a vehicle associated with the request.
- **Observed response fields:** Vehicle ID, vehicle-location ID, latitude, longitude, owner's full name and email address.

**Security notes:** The response contains location information and personal data. In this baseline test, the request concerned the vehicle registered to the test account. These fields should be treated as sensitive and must not be published with real or identifying values.

**Assessment status:** Normal behaviour observed for the test account. Cross-user authorisation has not yet been tested.

## 4. Authentication Test — Dashboard Without Authorization Header

### Objective

Determine how the dashboard endpoint responds when the request is sent without an Authorization header.

### Test Details

- **Method:** GET
- **Path:** `/identity/api/v2/user/dashboard`
- **Baseline with Authorization header:** HTTP 200
- **Test without Authorization header:** HTTP 404
- **Observed response message:** `Given Email is not registered`
- **Dashboard data returned:** No dashboard data was observed in the response

### Observed Behaviour

The authenticated request returned HTTP 200. After the Authorization header was removed, the request returned HTTP 404 with a message indicating that the email was not registered.

### Interpretation

The unauthenticated request produced a different result from the authenticated baseline. However, the reason for the 404 response has not been established.

The result alone does not prove that authentication is correctly enforced, nor does it demonstrate unauthorised access to dashboard data. Further testing is required to understand how the endpoint handles missing or invalid authentication.

### Limitations

The response was recorded from the local training application. The underlying application logic was not inspected, so the cause of the error remains unconfirmed.

### Follow-up Test — Invalid Authorization Token

**Objective:** Compare the dashboard endpoint's response when supplied with an invalid token.

**Test details:**
- **Method:** GET
- **Path:** `/identity/api/v2/user/dashboard`
- **Authorization header:** Present, with a deliberately invalid placeholder token
- **Observed status:** HTTP 404
- **Observed message:** `Given Email is not registered`
- **Dashboard data returned:** No dashboard data was observed

**Interpretation:**

The invalid-token request returned the same status code and message as the request without an Authorization header. Neither request returned dashboard data.

The matching responses suggest that the application handles these two test cases similarly, but they do not establish the precise internal reason for the response or prove that authentication controls are fully secure.

The result is recorded as an observation, not a confirmed vulnerability. Further testing would be needed to understand the endpoint's authentication behaviour.

**Security note:** The placeholder token was used only in the local training lab. Genuine tokens and session cookies must not be included in public evidence.

## BOLA Test — Cross-Account Vehicle Location

### Objective

Determine whether the vehicle-location endpoint checks that the authenticated user is authorised to access the requested vehicle.

### Test Details

- **Method:** GET
- **Endpoint:** `/identity/api/v2/vehicle/<vehicle-id>/location`
- **Authentication:** Bearer token belonging to the first test account
- **Test:** Changed the vehicle identifier to one associated with a different Community author in the local crAPI lab.
- **Observed status:** HTTP 200 OK
- **Observed result:** The response contained the other vehicle's location information and owner details.

### Interpretation

The endpoint returned information associated with a different vehicle while the request used the first test account's authentication token. This demonstrates Broken Object Level Authorisation (BOLA) behaviour in the tested training endpoint.

The result indicates that the endpoint did not enforce the expected object-level permission check for this request.

### Evidence

See the detailed report in [Finding 01 — BOLA in Vehicle Location API](../investigations/01-bola-vehicle-location.md).

The supporting screenshot is stored at `evidence/01-bola-request-response.png`.

### Remediation Summary

The server should verify that the authenticated user is authorised to access each requested vehicle. Knowing a vehicle identifier must not be sufficient to retrieve its location or owner information.

### Security and Privacy

Testing was limited to the local OWASP crAPI lab. Vehicle identifiers, coordinates, owner details and authentication tokens must not appear in public evidence.

## 5. Initial Observations

- The login flow returned an authentication token.
- An authentication-verification request returned HTTP 200.
- The authenticated dashboard request included an Authorization header and returned HTTP 200.
- Removing the Authorization header resulted in HTTP 404 and the message `Given Email is not registered`.
- The different responses provide a starting point for investigating authentication enforcement.
- No vulnerability has been confirmed from these observations alone.
- The vehicle-location endpoint returned location data and owner details for the vehicle registered to the authenticated test account.

## 6. Next Investigation Steps

1. Confirm that the authenticated and unauthenticated dashboard requests use the same method, host and path.
2. Verify that removing the Authorization header was the only intentional change to the request.
3. Compare the successful authenticated response with the unauthenticated response without recording or publishing sensitive data.
4. Investigate the behaviour of the endpoint when authentication is missing or invalid.
5. Assess authorisation controls using only the local crAPI environment and test accounts.
6. Document any confirmed findings with sanitised evidence, impact, remediation and retest criteria.

## 7. Evidence Handling

Do not publish passwords, Bearer tokens, session cookies, API keys or other authentication secrets. Any request or response screenshots must be sanitised before they are added to GitHub.
