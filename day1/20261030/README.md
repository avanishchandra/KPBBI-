# OTP Verification Bypass via HTTP Response Manipulation

## 1. Vulnerability Details

| Field | Details |
|---|---|
| Vulnerability Type | Authentication Bypass |
| Affected Functionality | Email and Mobile OTP Verification |
| Attack Vector | HTTP Response Manipulation |
| Testing Tool | Burp Suite Community Edition |
| Date | 02 November 2026 |
| Severity | To be determined |
| CWE | CWE-602 – Client-Side Enforcement of Server-Side Security |

## 2. Description

The application's registration workflow allows progression beyond email and mobile OTP verification through HTTP response manipulation.

During testing, invalid OTP submissions returned `401 Unauthorized` responses. After manipulating the intercepted responses using Burp Suite, the browser processed `201 Created` responses and displayed successful verification messages.

The registration workflow subsequently allowed access to the application dashboard without completing the normal OTP verification process.

This behaviour indicates a potential failure to independently enforce OTP verification on the server side.

## 3. Affected Endpoint

```http
POST /api/v1/uwa/auth/otp/verify
```

**Affected operations:**
- Email OTP verification
- Mobile OTP verification

*Production hostname withheld in this public report.*

## 4. Steps to Reproduce

1. Navigate to the application's account registration page.
2. Proceed to email OTP verification.
3. Submit an invalid OTP while monitoring traffic through Burp Suite.
4. Observe the original `401 Unauthorized` response.
5. Manipulate the intercepted response to return a successful verification result.
6. Observe the application's verification success message and continue registration.
7. Repeat the test during mobile OTP verification.
8. Observe successful progression to the application dashboard.

*Detailed target-specific reproduction evidence is withheld from this public copy.*

## 5. Expected Result

- The server should independently validate submitted OTPs.
- Invalid or expired OTPs must not result in successful verification.
- Registration should remain restricted until all required verification steps are completed.
- Client-side response manipulation must not grant verification privileges.

## 6. Actual Result

- Invalid OTP submissions returned `401 Unauthorized`.
- Manipulated responses were processed as `201 Created`.
- The application displayed successful email and mobile verification messages.
- Registration proceeded to the application dashboard.

## 7. Security Impact

If the backend does not independently enforce verification state, an attacker may:

- Register accounts using unverified email addresses.
- Bypass mobile number verification.
- Access functionality intended for users with verified contact details.

The impact on other application functionality depends on server-side verification enforcement.

## 8. Evidence

| Screenshot | Description |
|---|---|
| [01 – Original 401 Response](evidence/01-email-otp-401.png) | Invalid OTP response |
| [02 – Manipulated Email Response](evidence/02-email-otp-201.png) | Successful email verification response |
| [03 – Manipulated Mobile Response](evidence/03-mobile-otp-201.png) | Successful mobile verification response |
| [04 – Registration Success](evidence/04-registration-success.png) | Access to application dashboard |

## 9. Remediation

1. Enforce OTP validation exclusively through trusted server-side logic.
2. Maintain verification state on the server rather than relying on client-side responses.
3. Bind OTP challenges to the relevant user, contact identifier and registration session.
4. Enforce OTP expiration, single-use validation and attempt limits.
5. Require server-side verification checks before account activation and access to protected functionality.

---

**References**

- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [CWE-602: Client-Side Enforcement of Server-Side Security](https://cwe.mitre.org/data/definitions/602.html)
- [PortSwigger Web Security Academy – Authentication](https://portswigger.net/web-security/authentication)