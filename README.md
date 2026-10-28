# API Security Assessment — OWASP crAPI

## Project Overview

This project documents a practical API security assessment using OWASP crAPI, an intentionally vulnerable API training application running locally with Docker.

Using Burp Suite Community Edition, the assessment covers API reconnaissance, authentication behaviour, object-level authorisation, business-logic validation, property-level controls, administrative function authorisation and server-side request handling.

Each investigation records the methodology, observed behaviour, potential impact, evidence, remediation recommendations and retest criteria.

The project forms part of a cybersecurity portfolio focused on practical API penetration testing and professional vulnerability reporting.

## Objectives

- Identify API endpoints, HTTP methods and response codes.
- Inspect authentication flows and authenticated requests.
- Establish normal application behaviour before testing security controls.
- Investigate object-level authorisation for vehicle resources.
- Assess server-side validation of order quantities.
- Test whether users can modify internal API properties.
- Evaluate authorisation controls on administrative functions.
- Investigate server-side requests initiated using client-supplied URLs.
- Validate findings using repeatable requests and observable responses.
- Preserve sanitised evidence and protect authentication secrets.
- Recommend remediation and define retest criteria.

## Environment and Tools

- OWASP crAPI
- Docker Desktop and Docker Compose
- Burp Suite Community Edition
- PowerShell
- HTTP request and response analysis
- GitHub and Markdown reporting

The application runs locally in Docker. Published web and MailHog ports were checked to ensure they were bound to `127.0.0.1`. The identity service's host-mounted keys directory was configured as read-only.

## Assessment Documentation

### 1. API Endpoint Inventory and Authentication Observations

**Status:** Initial reconnaissance completed

Recorded endpoints observed during login, authentication verification, dashboard access, vehicle registration and vehicle-location requests.

The inventory includes HTTP methods, endpoint paths, observed responses, authentication context and initial security observations.

The dashboard returned HTTP 200 with the valid test account's token. Requests without an Authorization header and with an invalid token returned HTTP 404 with the message `Given Email is not registered`. The cause of those responses was not established, so they are documented as observations rather than confirmed vulnerabilities.

**Documentation:** [Read the API endpoint inventory](documentation/endpoint-inventory.md)

### 2. Broken Object Level Authorisation (BOLA) — Vehicle Location

**Status:** Confirmed in the local training environment

Used Burp Suite Repeater to change the vehicle identifier in a vehicle-location request while retaining the authenticated test account's Bearer token.

**Observed result:** The API returned HTTP 200 and included another vehicle's location information and owner details.

**Security significance:** The tested endpoint returned data associated with a different vehicle without enforcing the expected object-level permission check for the requesting test account.

**Investigation report:** [Read the BOLA vehicle-location finding](investigations/01-bola-vehicle-location.md)

**Supporting evidence:** [View the sanitised request and response](evidence/01-bola-request-response.png)

### 3. Negative Quantity Accepted by Shop Order API

**Status:** Confirmed in the local training environment

Used Burp Suite Repeater to change an order quantity to `-1` after establishing normal purchase behaviour.

**Observed result:** The API returned HTTP 200 with a successful order message. The available test-account credit increased from 90 to 100 following the negative-quantity request.

**Security significance:** The result demonstrates a business-logic validation flaw in the training application: a negative order quantity was accepted and the resulting transaction increased the available credit.

**Investigation report:** [Read the negative-quantity finding](investigations/02-negative-quantity-business-logic.md)

**Supporting evidence:** [View the sanitised request and response](evidence/02-negative-quantity-order.png)

### 4. User-Controlled Video Conversion Parameters

**Status:** Property modification confirmed; downstream impact not tested

Inspected the normal video-name update request, then added a client-supplied `conversion_params` property in Burp Suite Repeater.

**Observed result:** The API accepted the request and returned the updated `conversion_params` value.

**Security significance:** The behaviour demonstrates that the tested property could be modified through the API, even though it was not included in the normal video-name update request. This warrants a property-level authorisation review. Command execution or further downstream impact was not demonstrated.

**Investigation report:** [Read the video property modification finding](investigations/03-video-property-level-authorization.md)

**Supporting evidence:** [View the sanitised request and response](evidence/03-video-conversion-params.png)

### 5. Broken Function Level Authorisation (BFLA) — Administrative Video Deletion

**Status:** Confirmed in the local training environment

Used Account A's standard-user authentication token to submit a DELETE request to the administrative video endpoint, targeting a disposable test video belonging to Account B.

**Observed result:** The API returned HTTP 200 with the message `User video deleted successfully.`

**Security significance:** The administrative deletion operation accepted a request made using a standard user's token and reported successful deletion of another test user's video. This demonstrates an authorisation failure in the tested endpoint.

**Investigation report:** [Read the BFLA video-deletion finding](investigations/04-bfla-admin-video-deletion.md)

**Supporting evidence:** [View the sanitised request and response](evidence/04-bfla-admin-video-deletion.png)

### 6. Server-Side Request Forgery (SSRF) — Contact Mechanic API

**Status:** Server-side request behaviour demonstrated in the local training environment

Modified the `mechanic_api` field in a Contact Mechanic request to use the public destination `https://www.google.com`.

**Observed result:** The application returned a response from the external destination with HTTP 200.

**Security significance:** The result demonstrates that a client-supplied URL influenced a server-side HTTP request and that the resulting response was returned by the application. The test did not attempt to access internal services, cloud metadata endpoints or restricted network resources.

**Investigation report:** [Read the SSRF mechanic API finding](investigations/05-ssrf-mechanic-api.md)

**Supporting evidence:** [View the sanitised request and response](evidence/05-ssrf-mechanic-api.png)

## Reporting Methodology

The assessment follows a structured process:

1. Establish normal application behaviour.
2. Identify relevant endpoints, parameters and response fields.
3. Inspect HTTP requests, responses and authentication context.
4. Test security controls within the authorised local environment.
5. Record observed behaviour and supporting evidence.
6. Assess potential impact and document limitations.
7. Recommend remediation and define retest criteria.

Unexpected responses are recorded as observations rather than labelled vulnerabilities without sufficient supporting evidence. Conclusions are limited to the behaviour actually demonstrated.

## Scope and Safety

All testing is restricted to the local OWASP crAPI training application and its test data. No unrelated public APIs or production systems are tested.

Public reports and screenshots must exclude Bearer tokens, session cookies, passwords, VINs, PINs, vehicle identifiers, coordinates, personal names, email addresses, Base64 video data and other sensitive values.

The findings describe behaviour observed in this deliberately vulnerable training environment. They do not establish that another application has the same vulnerabilities.

## Skills Demonstrated

- API reconnaissance and endpoint inventory
- Authentication-flow analysis
- HTTP request and response inspection
- Burp Suite Proxy and Repeater
- Object-level authorisation testing
- Business-logic and input-validation testing
- API property modification testing
- Administrative function authorisation testing
- Server-side request handling assessment
- Evidence collection and sanitisation
- Vulnerability impact assessment
- Remediation and retest planning
- Technical documentation and reporting
