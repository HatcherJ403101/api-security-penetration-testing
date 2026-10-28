# Finding 05 — Server-Side Request Forgery in Contact Mechanic API

## 1. Assessment Overview

**Application:** OWASP crAPI  
**Environment:** Local Docker training lab  
**Testing tool:** Burp Suite Community Edition  
**Endpoint:** `POST /workshop/api/merchant/contact_mechanic`  
**Vulnerability category:** Server-Side Request Forgery (SSRF)  
**Test destination:** `https://www.google.com`  
**Observed result:** The application returned content from the supplied external URL with status 200.

## 2. Objective

Determine whether the Contact Mechanic API makes server-side HTTP requests to a client-supplied URL and returns the resulting response.

## 3. Methodology

1. Signed in to the local crAPI application using a dedicated test account.
2. Submitted a normal Contact Mechanic request through the application interface.
3. Captured the request in Burp Suite and identified the `mechanic_api` field.
4. Established normal request behaviour before changing the URL.
5. Modified only the `mechanic_api` value to `https://www.google.com`, retaining the other required fields and the existing authentication context.
6. Submitted the request once.
7. Observed that the application returned a response from the external destination with status 200.

Testing was restricted to the local, intentionally vulnerable crAPI training environment. The URL used for this proof-of-concept was a benign public destination.

## 4. Finding and Evidence

### Expected Behaviour

Applications that fetch user-supplied URLs should validate and restrict the destination to explicitly permitted hosts and schemes. The server should not blindly retrieve arbitrary resources on behalf of a user.

### Observed Behaviour

The Contact Mechanic endpoint accepted an externally supplied URL through the `mechanic_api` field. When the field was set to `https://www.google.com`, the application returned content from that destination with status 200.

This demonstrates user-controlled server-side HTTP request behaviour consistent with an SSRF vulnerability in the training application.

### Evidence

See `evidence/05-ssrf-mechanic-api.png` for the sanitised request and response screenshot.

The screenshot must not expose Bearer tokens, cookies, VINs or other sensitive values.

## 5. Security Impact

If an application fetches arbitrary user-supplied URLs without appropriate restrictions, an attacker may be able to influence requests made from the server's network position.

Depending on the application environment and network controls, this type of weakness can potentially expose internal services or resources that are not directly accessible to external users.

This assessment confirmed a server-side request to a public destination. It did not test access to internal services, cloud metadata endpoints or other restricted resources, and no such impact is claimed.

## 6. Risk Assessment

**Assessment:** SSRF behaviour demonstrated in the local training environment.

The test established that the application made a request using a client-supplied URL and returned the resulting external response.

Real-world severity would depend on the destinations the server can reach, the privileges available to the server, and the effectiveness of network and application-level restrictions. No formal CVSS score has been assigned.

## 7. Recommended Remediation

- Avoid accepting complete user-supplied URLs where a fixed destination or allowlisted identifier is sufficient.
- Restrict permitted destinations to an explicit allowlist of approved hosts and schemes.
- Resolve hostnames and validate destination IP addresses before making requests.
- Block loopback, private, link-local and other prohibited destination ranges, including after redirects and DNS resolution.
- Disable redirects or validate every redirect destination.
- Apply egress network controls so the application can contact only the services it requires.
- Use connection timeouts, response-size limits and safe error handling.
- Add regression tests for allowed destinations, disallowed destinations and redirect behaviour.

## 8. Retest Criteria

After remediation:

1. Confirm that legitimate Contact Mechanic requests continue to work.
2. Verify that only approved destination URLs are accepted.
3. Confirm that requests to disallowed destinations are rejected before a connection is made.
4. Check that redirects cannot bypass destination restrictions.
5. Verify that the API does not expose unintended response content from a disallowed destination.

## 9. Limitations

- Testing was performed only against the local OWASP crAPI training environment.
- The proof-of-concept used the public destination `https://www.google.com`.
- No attempt was made to access internal services, cloud metadata or restricted network resources.
- No production application or third-party system was targeted for exploitation.
- The finding describes the observed behaviour and does not claim untested downstream impact.

## 10. Conclusion

The Contact Mechanic API accepted a client-supplied external URL and returned the resulting HTTP response.

This demonstrates SSRF behaviour in the local training application. The report records the methodology, observed behaviour, potential impact, limitations, remediation and retest criteria.

## 11. Evidence

### Evidence 05 — SSRF Request and Response

[View sanitised request and response](../evidence/05-ssrf-mechanic-api.png)

The screenshot shows the Contact Mechanic API request with `mechanic_api` set to `https://www.google.com` and the corresponding HTTP 200 response.

This evidence supports the observation that the application made a server-side request to a client-supplied external URL and returned the resulting response.

The published screenshot must redact Bearer tokens, cookies, VINs and other sensitive values. The assessment did not test access to internal services or restricted network resources.
