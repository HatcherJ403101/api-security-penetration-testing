# Finding 04 — Broken Function Level Authorisation in Video Deletion

## 1. Assessment Overview

**Application:** OWASP crAPI  
**Environment:** Local Docker training lab  
**Testing tool:** Burp Suite Community Edition  
**Endpoint:** `DELETE /identity/api/v2/admin/videos/{video_id}`  
**Vulnerability category:** Broken Function Level Authorisation (BFLA)  
**Observed status:** HTTP 200 OK

## 2. Objective

Determine whether a standard authenticated user could invoke an administrative video-deletion function to delete a video belonging to a different test account.

## 3. Methodology

1. Created two test accounts in the local crAPI environment.
2. Uploaded a separate test video under each account.
3. Identified the video ID from Account B's upload response.
4. Captured a normal request associated with Account A in Burp Suite.
5. Constructed a DELETE request to the administrative video endpoint.
6. Used Account A's ordinary-user Bearer token and Account B's video ID.
7. Sent the request once and recorded the response.

All testing was restricted to the local, intentionally vulnerable training environment.

## 4. Finding and Evidence

### Expected Behaviour

Administrative video-deletion functionality should be available only to authorised administrators. A standard user should not be able to invoke an administrative endpoint simply by changing the request path or HTTP method.

### Observed Behaviour

The API returned HTTP 200 with the message:

`User video deleted successfully.`

The request used Account A's authentication token to target Account B's test video.

The response indicates that the application accepted the administrative deletion request from the standard test account. The result is consistent with a missing or ineffective server-side function-level authorisation check.

### Evidence

See `evidence/04-bfla-admin-video-deletion.png` for the sanitised request and response screenshot.

The screenshot must not expose authentication tokens, session cookies, or the actual video identifier.

## 5. Security Impact

An ordinary authenticated user could invoke an administrative function and delete another user's video in the training application.

In a real application, this type of flaw could allow unauthorised users to delete or modify other users' resources or access other privileged operations. The actual impact depends on which administrative functions are affected and the resources those functions control.

This assessment confirmed the deletion behaviour for one test video. It did not test other administrative endpoints.

## 6. Risk Assessment

**Assessment:** Confirmed BFLA behaviour in the local training environment.

The API accepted the administrative deletion request made with a standard user's token and returned a successful deletion message.

The cross-account target also demonstrates an object-level authorisation failure for this operation. No formal CVSS score has been assigned.

## 7. Recommended Remediation

- Enforce server-side role and permission checks for every administrative endpoint.
- Verify the authenticated user's privileges before executing any deletion.
- Ensure ordinary users can delete only resources they are authorised to manage through user-level endpoints.
- Do not rely on endpoint paths, HTTP methods or client-side controls as a substitute for authorisation.
- Apply consistent authorisation rules across all administrative functions.
- Add automated tests confirming that standard users receive an appropriate denial when invoking administrator-only operations.

## 8. Retest Criteria

After remediation:

1. Authenticate as a standard test user and attempt the administrative deletion operation against a disposable test resource.
2. Confirm the API denies the operation and does not delete the resource.
3. Verify that an authorised administrator can perform the intended operation.
4. Confirm that ordinary users can still manage their own resources using the intended user-level functionality.
5. Review related administrative endpoints for similar authorisation failures.

## 9. Limitations

- Testing was conducted only against the local OWASP crAPI training application.
- The test covered one administrative video-deletion endpoint.
- The evidence establishes the observed deletion behaviour but does not establish the security of every administrative endpoint.
- No public service or real user's data was targeted.
- Published evidence must exclude authentication secrets and unnecessary identifiers.

## 10. Conclusion

The administrative video-deletion endpoint accepted a request authenticated as a standard user and returned a successful deletion message for a video belonging to a different test account.

The observed behaviour demonstrates a BFLA vulnerability in the tested training endpoint, alongside a cross-user resource authorisation failure. The report documents the methodology, evidence, impact, remediation and retest criteria.
