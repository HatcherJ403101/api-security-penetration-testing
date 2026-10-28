# Finding 02 — Negative Quantity Accepted by Shop Order API

## 1. Assessment Overview

**Application:** OWASP crAPI  
**Environment:** Local Docker training lab  
**Testing tool:** Burp Suite Community Edition  
**Endpoint:** `POST /workshop/api/shop/orders`  
**Vulnerability category:** Business Logic Validation / Negative Quantity Handling  
**Observed status:** HTTP 200 OK

## 2. Objective

Determine whether the shop API validates order quantities and prevents negative values from being used to manipulate the account credit balance.

## 3. Methodology

1. Authenticated using a dedicated test account in the local crAPI application.
2. Placed a normal order through the Shop interface.
3. Observed that the normal order reduced the available credit from 100 to 90.
4. Captured the order request in Burp Suite and sent it to Repeater.
5. Changed the quantity from its normal positive value to `-1`, keeping the product identifier and authentication context unchanged.
6. Submitted the modified request to the local training API.
7. Recorded the HTTP status, response message, order ID progression and resulting account credit.

All testing was performed within the local, deliberately vulnerable crAPI environment.

## 4. Finding and Evidence

### Expected Behaviour

The API should reject negative order quantities and validate order data on the server before creating an order or updating account credit.

### Observed Behaviour

The modified request was accepted with HTTP 200 OK. The response indicated that the order had been sent successfully and returned an incremented order ID.

The available credit increased from 90 after the normal purchase to 100 after the negative-quantity request.

This demonstrates that the API accepted the negative quantity and that the resulting transaction increased the available credit rather than deducting it.

### Evidence

See `evidence/02-negative-quantity-order.png` for the sanitised request and response evidence.

The screenshot must not expose authentication tokens, cookies or other secrets.

## 5. Security Impact

The finding demonstrates that an authenticated user can manipulate the order quantity to increase credit within the training application.

In a real application, similar behaviour could undermine transaction integrity and allow an attacker to manipulate account balances, store credit or purchase calculations. The actual impact would depend on the application's financial logic, validation controls and whether the balance represents real monetary value.

No real financial transaction was involved in this assessment.

## 6. Risk Assessment

**Assessment:** Confirmed negative-quantity business-logic flaw in the training application.

The API accepted a negative quantity and the available credit increased instead of decreasing.

The real-world severity would depend on the value represented by the balance and the extent to which the resulting credit could be used. No formal CVSS score has been assigned.

## 7. Recommended Remediation

- Enforce server-side validation requiring order quantities to be positive integers.
- Reject zero, negative, non-integer and otherwise invalid quantities where they are outside the business rules.
- Validate quantity values before creating an order or updating a balance.
- Calculate transaction values using trusted server-side product prices and validated quantities.
- Ensure failed validation leaves both the order history and account balance unchanged.
- Add regression tests covering negative quantities, zero quantities and normal positive quantities.

Client-side form validation alone is insufficient because an attacker can modify requests outside the normal interface.

## 8. Retest Criteria

After remediation:

1. Submit an ordinary positive quantity and confirm the order and balance are correct.
2. Submit a negative quantity and confirm that the API rejects it.
3. Verify that a rejected order does not create a completed order or change the account balance.
4. Check that invalid requests receive an appropriate error response.
5. Confirm that legitimate purchases continue to work correctly.

## 9. Limitations

- Testing was confined to the local OWASP crAPI training environment.
- The test used a negative quantity of `-1`.
- No attempt was made to maximise the credit increase or exploit a real payment system.
- The test demonstrates the observed behaviour but does not establish the impact on a production financial system.
- Public evidence must exclude authentication secrets.

## 10. Conclusion

The assessment demonstrated that the shop order endpoint accepted a negative quantity, returned a successful response and increased the available credit in the training application.

The finding highlights the importance of server-side business-rule validation and correct transaction calculations. The report records the observed behaviour, potential impact, remediation recommendations and retest criteria.
