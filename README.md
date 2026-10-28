# API Security Assessment — OWASP crAPI

## Project Overview

This project documents a practical API security assessment using OWASP crAPI, an intentionally vulnerable API training application, running locally with Docker.

The project focuses on API reconnaissance, authentication behaviour, vehicle-related endpoints, object-level authorisation and evidence-based vulnerability reporting. Testing is performed using Burp Suite Community Edition and a dedicated local test account.

The aim is to develop practical API penetration-testing skills and produce clear technical reports containing observed behaviour, potential impact, remediation recommendations and retest criteria.

## Objectives

- Identify API endpoints, HTTP methods and response codes.
- Observe normal authentication and authenticated API workflows.
- Inspect HTTP requests and responses using Burp Suite.
- Establish baseline behaviour before testing security controls.
- Investigate object-level authorisation for vehicle resources.
- Validate findings using repeatable requests and observable responses.
- Document evidence while protecting authentication secrets and personal data.
- Recommend practical remediation and define retest criteria.

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

Recorded the login, authentication-verification, dashboard, vehicle-registration and vehicle-location endpoints observed during normal use of the local test account.

The inventory documents HTTP methods, endpoint paths, observed responses, authentication context and initial security observations.

**Documentation:** [Read the API endpoint inventory](documentation/endpoint-inventory.md)

### 2. Authentication Behaviour — Dashboard Endpoint

**Status:** Initial observations recorded

Compared the authenticated dashboard request with requests sent without an Authorization header and with a deliberately invalid token.

- Authenticated request: HTTP 200
- Request without Authorization header: HTTP 404, `Given Email is not registered`
- Request with invalid token: HTTP 404, `Given Email is not registered`

The matching error responses did not establish the precise cause or independently prove whether authentication controls were secure. These outcomes are documented as observations, not confirmed vulnerabilities.

### 3. Broken Object Level Authorisation (BOLA) — Vehicle Location

**Status:** Confirmed in the local training environment

Tested the vehicle-location endpoint using the first test account's Bearer token, then changed the vehicle identifier to one associated with a different Community author.

**Observed result:** The endpoint returned HTTP 200 and included the other vehicle's location information and owner details.

**Security significance:** The observed behaviour demonstrates BOLA in the tested training endpoint because object-level access was not restricted as expected for the requesting test account.

**Investigation report:** [Read the BOLA vehicle-location finding](investigations/01-bola-vehicle-location.md)

**Supporting evidence:** [View the sanitised request and response](evidence/01-bola-request-response.png)

## Reporting Methodology

The assessment follows a structured process:

1. Establish normal application behaviour.
2. Identify relevant API endpoints and their parameters.
3. Inspect requests, responses and authentication context.
4. Test security controls within the authorised local environment.
5. Record observed behaviour and supporting evidence.
6. Assess potential impact and document limitations.
7. Recommend remediation and define retest criteria.

Findings are based on observed results. Unexpected responses are not classified as vulnerabilities without sufficient supporting evidence.

## Scope and Safety

All testing is restricted to the local OWASP crAPI training application and its test data. No unrelated public APIs or production systems are tested.

Public reports and screenshots must exclude Bearer tokens, session cookies, passwords, VINs, PINs, vehicle identifiers, coordinates, personal names, email addresses and other sensitive values.

The findings describe behaviour observed in this training environment and do not imply that another application has the same vulnerability.

## Skills Demonstrated

- API reconnaissance and endpoint inventory
- Authentication-flow analysis
- HTTP request and response inspection
- Burp Suite Proxy and Repeater
- API access-control testing
- BOLA identification and validation
- Evidence handling and sanitisation
- Vulnerability impact assessment
- Remediation and retest planning
- Technical documentation and reporting
