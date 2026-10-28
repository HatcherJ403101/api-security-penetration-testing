# API Security Assessment — OWASP crAPI

## Project Overview

This project documents a practical API security assessment using OWASP crAPI, an intentionally vulnerable API training application running locally with Docker.

The assessment uses Burp Suite Community Edition to examine HTTP requests and responses, establish normal application behaviour, test API security controls, and document validated findings with supporting evidence and remediation recommendations.

The project forms part of a cybersecurity portfolio focused on developing practical API penetration-testing skills and professional technical reporting.

## Objectives

- Identify API endpoints, HTTP methods and response codes.
- Inspect authentication flows and authenticated requests.
- Establish baseline behaviour before testing security controls.
- Investigate object-level authorisation for vehicle resources.
- Assess server-side validation of order quantities.
- Validate findings using controlled, repeatable requests.
- Record sanitised evidence while protecting sensitive data.
- Explain security impact and recommend practical remediation.
- Define retest criteria for validating security fixes.

## Environment and Tools

- OWASP crAPI
- Docker Desktop and Docker Compose
- Burp Suite Community Edition
- PowerShell
- HTTP request and response analysis
- GitHub and Markdown reporting

The application runs locally in Docker. Published web and MailHog ports were checked to ensure they were bound to `127.0.0.1`. The identity service's host-mounted keys directory was configured as read-only.

## Assessment Documentation

### 1. API Endpoint Inventory

**Status:** In progress

Recorded endpoints observed during normal login, authentication verification, dashboard access, vehicle registration and vehicle-location requests.

The inventory documents HTTP methods, paths, observed responses, authentication context and initial security observations.

**Documentation:** [Read the API endpoint inventory](documentation/endpoint-inventory.md)

### 2. Broken Object Level Authorisation (BOLA) — Vehicle Location

**Status:** Confirmed in the local training environment

Tested the vehicle-location endpoint using the first test account's Bearer token, then changed the vehicle identifier to one associated with a different Community author.

**Observed result:** The API returned HTTP 200 and included the other vehicle's location information and owner details.

**Security significance:** The tested endpoint returned another vehicle's information without enforcing the expected object-level permission check for the requesting test account.

**Investigation report:** [Read the BOLA vehicle-location finding](investigations/01-bola-vehicle-location.md)

**Supporting evidence:** [View the sanitised request and response](evidence/01-bola-request-response.png)

### 3. Negative Quantity Accepted by Shop Order API

**Status:** Confirmed in the local training environment

Examined the shop-order endpoint using Burp Suite Repeater. After establishing normal purchase behaviour, changed the order quantity to `-1` while keeping the product identifier and authentication context unchanged.

**Observed result:** The API returned HTTP 200 and a successful order message. The test account's available credit increased from 90 to 100 following the negative-quantity request.

**Security significance:** The behaviour demonstrates a business-logic validation flaw in the training application. The API accepted an invalid negative quantity and the resulting transaction increased the available credit.

**Investigation report:** [Read the negative-quantity finding](investigations/02-negative-quantity-business-logic.md)

**Supporting evidence:** [View the sanitised request and response](evidence/02-negative-quantity-order.png)

## Reporting Methodology

The assessment follows a structured process:

1. Establish normal application behaviour.
2. Identify endpoints and relevant request parameters.
3. Inspect HTTP requests, responses and authentication context.
4. Test security controls within the authorised local environment.
5. Record observed behaviour and supporting evidence.
6. Assess potential impact and document limitations.
7. Recommend remediation and define retest criteria.

Unexpected responses are recorded as observations rather than classified as vulnerabilities without sufficient supporting evidence.

## Scope and Safety

All testing is restricted to the local OWASP crAPI training application and its test data. No unrelated public APIs or production systems are tested.

Public reports and screenshots must exclude Bearer tokens, session cookies, passwords, VINs, PINs, vehicle identifiers, coordinates, personal names, email addresses and other sensitive values.

The findings describe behaviour observed in this deliberately vulnerable training environment. They do not establish that other applications have the same vulnerabilities.

## Skills Demonstrated

- API reconnaissance and endpoint inventory
- Authentication-flow analysis
- HTTP request and response inspection
- Burp Suite Proxy and Repeater
- API access-control testing
- BOLA identification and validation
- Business-logic and input-validation testing
- Evidence collection and sanitisation
- Vulnerability impact assessment
- Remediation and retest planning
- Technical documentation and reporting
