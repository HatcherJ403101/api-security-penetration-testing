# Finding 03 — User-Controlled Video Conversion Parameters

## 1. Assessment Overview

**Application:** OWASP crAPI  
**Environment:** Local Docker training lab  
**Testing tool:** Burp Suite Community Edition  
**Endpoint:** `PUT /identity/api/v2/user/videos/{video_id}`  
**Vulnerability category:** Broken Object Property Level Authorisation (BOPLA) / Mass Assignment

## 2. Objective

Determine whether the video-update endpoint accepts changes to an internal video-processing property that is not exposed by the ordinary video-name editing interface.

## 3. Methodology

1. Uploaded a harmless test video through the local crAPI interface.
2. Inspected the upload response and observed the `conversion_params` property.
3. Used the ordinary video-name editing interface to establish the normal update request.
4. Captured the resulting PUT request in Burp Suite.
5. Observed that the normal request body contained the video-name field.
6. Sent the request to Repeater and added a `conversion_params` property with the value `-v codec h265`.
7. Submitted the modified request and inspected the response.
8. Confirmed that the response returned the updated `conversion_params` value.

Testing was restricted to the local, intentionally vulnerable crAPI training environment.

## 4. Finding and Evidence

### Expected Behaviour

The API should explicitly define which properties an ordinary user can modify and validate updates against that allowlist. Internal processing properties should not be modifiable unless the operation is intentionally authorised and safely designed.

### Observed Behaviour

The normal video-editing interface submitted a video-name update. After adding a `conversion_params` property to the API request, the server accepted the value and returned the updated property in the response.

This demonstrates that the API permits client-controlled modification of the tested property.

### Evidence

See `evidence/03-video-conversion-params.png` for the sanitised Repeater request and response.

Authentication tokens and Base64 video data must not be published.

## 5. Security Impact

Allowing an API client to modify an internal processing property may expose functionality beyond what the ordinary interface intends to offer.

The potential impact depends on how the property is subsequently used by the application. In particular, internal processing parameters should not be trusted merely because they were supplied through an authenticated request.

This investigation confirmed property modification only. It did not demonstrate command execution, server compromise or access to other users' data.

## 6. Risk Assessment

**Assessment:** User-controlled modification of an internal video property confirmed in the local training environment.

The severity of an equivalent production issue would depend on the purpose of the property, the permissions available to the user, and the way downstream services consume the value.

No formal CVSS score has been assigned.

## 7. Recommended Remediation

- Allowlist the properties that ordinary users can modify through the video-update endpoint.
- Separate user-editable metadata from internal video-processing configuration.
- Avoid automatically binding arbitrary client-supplied properties to internal application objects.
- Validate property types and permitted values on the server.
- Ensure downstream processing does not interpret user-controlled properties as executable instructions.
- Add regression tests proving that unauthorised properties are rejected or ignored.
- Review existing video records and processing workflows for unexpected property values.

## 8. Retest Criteria

After remediation:

1. Update the video name through the normal interface and confirm that the operation still succeeds.
2. Submit an additional request containing the internal property.
3. Confirm that the property cannot be changed by an ordinary user unless the functionality is explicitly authorised.
4. Verify that normal video processing continues to work.
5. Confirm that validation failures do not cause unintended changes to the video record.

## 9. Limitations

- The assessment was conducted only against the local OWASP crAPI training environment.
- Testing confirmed that the API accepted a modified `conversion_params` value.
- No command-execution payload was tested.
- No production application or third-party system was accessed.
- Public evidence must exclude authentication tokens and unnecessary video data.

## 10. Conclusion

The video-update endpoint accepted a client-supplied `conversion_params` property that was not present in the ordinary video-name update request.

This demonstrates user-controlled modification of an internal video property in the training application. The report records the observed behaviour, potential impact, remediation and retest criteria without claiming untested downstream effects.
