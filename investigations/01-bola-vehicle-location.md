# Finding 01 — Broken Object Level Authorisation (BOLA) in Vehicle Location API

## 1. Assessment Overview

**Application:** OWASP crAPI  
**Environment:** Local Docker training lab  
**Testing tool:** Burp Suite Community Edition  
**Endpoint:** `GET /identity/api/v2/vehicle/<vehicle-id>/location`  
**Vulnerability category:** Broken Object Level Authorisation (BOLA)  
**Observed status:** HTTP 200 OK

This assessment examined whether the vehicle-location endpoint checked that the authenticated user was authorised to access the requested vehicle.

## 2. Objective

Determine whether an authenticated test user could retrieve vehicle-location and owner information associated with a different account by changing the vehicle identifier in the request path.

## 3. Methodology

1. Registered and authenticated a dedicated test account in the local crAPI environment.
2. Added a vehicle using the training data supplied by the local application.
3. Captured the vehicle-location request in Burp Suite.
4. Sent the request to Repeater and established the normal response for the test user's own vehicle.
5. Inspected the local Community API response to identify a vehicle associated with a different author.
6. Replayed the vehicle-location request while authenticated as the original test user, changing only the vehicle identifier in the URL path.
7. Inspected the response for evidence of unauthorised data access.

Testing was restricted to the local, deliberately vulnerable crAPI lab. No public API or production account was targeted.

## 4. Finding and Evidence

### Expected Behaviour

The server should verify that the authenticated user is authorised to access the requested vehicle before returning its location or associated owner information.

Knowing or supplying another vehicle's identifier should not grant access to its data.

### Observed Behaviour

The request using the authenticated test user's token returned HTTP 200 OK after the vehicle identifier was changed to one associated with a different Community author.

The response contained the other vehicle's location information and owner details.

This demonstrates that the endpoint returned another account's object data without enforcing the expected object-level authorisation control for the tested request.

### Vulnerability Classification

**Broken Object Level Authorisation (BOLA).**

The issue is also commonly described as an insecure direct object reference (IDOR) when an attacker can manipulate an object identifier to access information that should be restricted.

### Evidence Handling

Evidence should demonstrate the request structure, the changed vehicle identifier and the successful response. Public screenshots and reports must redact:

- Vehicle identifiers
- Latitude and longitude values
- Owner names and email addresses
- Bearer tokens and session cookies
- VINs, PINs and any other secrets

Do not publish the actual identifiers or sensitive response values.

## 5. Security Impact

An authenticated user could potentially retrieve vehicle-location information and associated owner details belonging to another user.

In a real application, unauthorised disclosure of location data could create privacy and safety concerns. Exposing owner information could also reveal personal data.

The impact demonstrated in this assessment is limited to the data returned by the tested endpoint in the training application. No additional account actions, location tracking over time or further access were tested.

## 6. Risk Assessment

**Assessment:** Confirmed BOLA behaviour in the training environment.

The endpoint returned another vehicle's data to an authenticated user who was not established as authorised to access that vehicle.

The real-world severity would depend on the sensitivity of the data, the number of affected users, the ease of discovering vehicle identifiers, and any additional controls. No formal CVSS score has been assigned.

## 7. Recommended Remediation

- Enforce object-level authorisation on every request that accesses vehicle data.
- Derive the requesting user's identity from the validated authentication context.
- Check that the requested vehicle belongs to the user or that the user has another explicitly authorised relationship to it.
- Deny access when the requester is not authorised, using an appropriate error response.
- Apply the same authorisation checks consistently across vehicle details, location data and related API endpoints.
- Add automated tests that attempt to access one test user's vehicle using a different test user's session.
- Review relevant API access logs for unexpected cross-account vehicle access where appropriate.

Using unpredictable vehicle identifiers or hiding identifiers in the client is not a substitute for server-side authorisation.

## 8. Retest Criteria

After remediation:

1. Request the location of a vehicle owned by the authenticated test user and confirm access works as intended.
2. Attempt to request a different test user's vehicle using the same account and authentication token.
3. Confirm that the second request does not disclose the vehicle location or owner information.
4. Verify that authorised workflows continue to function.
5. Repeat the check for other endpoints that expose vehicle-related data.

## 9. Limitations

- The test was conducted only against the local OWASP crAPI training environment.
- The observed HTTP status was 200 OK.
- The test established disclosure of another vehicle's location and owner details through the tested endpoint.
- No production application or real person's data was accessed.
- The assessment did not establish the impact of any additional API actions.
- Public evidence must be sanitised to avoid exposing identifiers, coordinates or authentication secrets.

## 10. Conclusion

The vehicle-location endpoint returned another author's vehicle-location information and owner details when the authenticated test user changed the vehicle identifier in the request path.

The observed behaviour demonstrates a BOLA vulnerability in the tested training endpoint. The report records the reproduction method, observed impact, limitations, recommended remediation and retest criteria.

This investigation demonstrates practical API testing with Burp Suite, object-level authorisation analysis, evidence handling and technical vulnerability reporting.

## 11. Evidence

### Evidence 01 — BOLA Request and Response

[View sanitised request and response screenshot](../evidence/01-bola-request-response.png)

The combined screenshot shows the modified vehicle-location request and the HTTP 200 response from the local OWASP crAPI training environment.

The evidence supports the observation that the endpoint returned another vehicle's location and owner information while the request used the authenticated test account's token.

Sensitive information, including the Bearer token, vehicle identifier, coordinates, owner's name and email address, has been redacted from the public evidence.
