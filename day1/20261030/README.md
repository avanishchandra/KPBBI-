# OTP Verification Bypass via HTTP Response Manipulation

## Summary

The registration workflow allowed progression past both email and mobile OTP verification after the corresponding HTTP responses were manipulated in Burp Suite.

The application initially returned `401 Unauthorized` with an `OTP invalid or expired` message. After response manipulation, the browser received a success-shaped response (`201 Created`), displayed successful verification feedback, and allowed registration to proceed to the authenticated application dashboard.

This indicates that the registration workflow may rely on client-side handling of OTP verification responses rather than independently enforcing verification state on the server for subsequent account access.

## Affected Functionality

- Email OTP verification during registration
- Mobile OTP verification during registration
- Registration completion and access to the application dashboard

**Target:** Production web application (service identifier and endpoint withheld in this public copy pending coordinated disclosure).

## Vulnerability

**Category:** Authentication / Improper Enforcement of Verification  
**Testing tool:** Burp Suite Community Edition  
**Date observed:** 02 November 2026  
**Severity:** To be determined by the affected service's security team after validating the server-side impact.

## Steps to Reproduce

> The detailed target-specific reproduction procedure is intentionally withheld from this public portfolio copy. It should be provided privately to the service owner through an authorised vulnerability disclosure channel.

At a high level, the observed test sequence was:

1. Begin the account registration workflow and reach email OTP verification.
2. Submit an invalid or expired OTP while observing the traffic in Burp Suite.
3. Observe the application's unsuccessful verification response.
4. Manipulate the intercepted response so the client processes a successful verification result.
5. Continue registration and repeat the observation for mobile OTP verification.
6. Observe that the application displays successful verification messages and permits access to the application dashboard.

The test was performed against the application represented in the attached evidence. Do not reproduce these actions against systems without explicit authorisation.

## Expected Result

- The server must independently validate each OTP.
- An invalid, expired, reused, or unverified OTP must not change the verification state.
- Registration and protected application access must remain blocked until the server confirms the required verification steps.
- Client-side response changes must not grant or persist verification privileges.

## Actual Result

- The original OTP verification response indicated failure with `401 Unauthorized`.
- Following response manipulation, the browser processed a success-shaped `201 Created` response.
- The application displayed successful email and mobile verification feedback.
- Registration proceeded to the application dashboard.

The observed behaviour indicates that the client-side registration flow can be advanced by manipulating verification responses. The service owner should verify whether the backend also records the email and mobile identifiers as verified and whether protected API operations enforce that state.

## Security Impact

If verification state is trusted from the client rather than enforced by the backend, an attacker may be able to:

- Complete registration without controlling the email address or mobile number supplied.
- Create accounts with unverified contact details.
- Bypass workflows that depend on verified contact information.

The impact on downstream functions depends on whether server-side APIs independently enforce verification state.

## Evidence

Only the essential, redacted screenshots are included.

| Evidence | Description |
|---|---|
| [01 – Original email OTP response](evidence/01-email-otp-original-401.png) | The application returns `401 Unauthorized` and an invalid/expired OTP message. |
| [02 – Email OTP success response](evidence/02-email-otp-response-201.png) | A success-shaped `201 Created` response observed after response manipulation. |
| [03 – Mobile OTP success response](evidence/03-mobile-otp-response-201.png) | A success-shaped response observed in the mobile verification flow. |
| [04 – Registration dashboard](evidence/04-registration-dashboard.png) | The application dashboard is accessible and displays mobile verification success feedback. |

Personal identifiers and target-specific request details have been redacted.

## Recommended Remediation

1. **Enforce verification server-side.** Store verification state in a server-controlled session or database record. Never accept a client-provided success flag as proof of OTP validation.
2. **Bind OTP challenges to the account and purpose.** Associate each challenge with the intended user, contact identifier, registration session, and verification purpose.
3. **Validate every protected operation.** Registration completion and protected API endpoints must independently check the required verification state.
4. **Use single-use, expiring OTPs.** Invalidate an OTP after successful use and enforce a short expiry period.
5. **Apply attempt limits.** Rate-limit OTP generation and verification attempts, with appropriate abuse monitoring.
6. **Test tampering resistance.** Add server-side tests confirming that modified client responses, altered frontend state, or replayed requests cannot mark a contact as verified.

## References

- [OWASP Web Security Testing Guide – Identity Management Testing](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [PortSwigger Web Security Academy – Authentication](https://portswigger.net/web-security/authentication)
- [RFC 9110 – HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)

---

**Disclosure note:** This public case study omits the production hostname, exact endpoint, and detailed manipulation instructions. Share the complete request/response evidence and reproducible details privately with the service owner through an authorised disclosure process. Do not publish unredacted traffic or personal data.
