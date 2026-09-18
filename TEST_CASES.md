# Consolidated Test Cases

## 1. Purpose

This file consolidates the test-case inventories used across the project evaluation. It is intended to support the final academic report without duplicating the detailed per-phase reports.

The complete detailed case sets remain in:

- [TEST_CASES_BLACKBOX.md](TEST_CASES_BLACKBOX.md)
- [TEST_CASES_WHITEBOX.md](TEST_CASES_WHITEBOX.md)
- [TEST_CASES_AUTOMATED.md](TEST_CASES_AUTOMATED.md)
- [TEST_CASES_ADVANCED.md](TEST_CASES_ADVANCED.md)

## 2. Test Case Categories

### Black-box cases

Black-box cases evaluated endpoints and user-visible flows without relying on internal code structure. Representative categories included:

- login and logout,
- registration,
- search and product lookup,
- basket and checkout,
- authenticated profile and upload flows,
- direct HTTP response checks,
- security header inspection.

### White-box cases

White-box cases focused on internal backend logic and branch conditions, especially in:

- [lib/insecurity.ts](lib/insecurity.ts)
- [lib/utils.ts](lib/utils.ts)

Representative focused checks:

- `isDeluxe` without token returns false,
- valid unknown cookie token is restored,
- invalid cookie token is ignored,
- environment mismatch remains enabled in `auto` mode.

### Automated cases

Automated tests used the repository’s native test infrastructure. The executed automation included:

- server unit suite,
- API integration suite,
- frontend unit suite,
- security regression subset,
- HTTP header regression suite.

### Advanced cases

Advanced cases were derived from risk-based security review and existing test infrastructure. These covered:

- authentication and authorization edge conditions,
- JWT validation and malformed tokens,
- injection and search payload behavior,
- profile upload and file content validation,
- error handling and response leakage,
- security headers and CORS behavior,
- rate-limit configuration review.

## 3. Executed Case Status

The following status is based on actual execution performed in this project phase.

| Category | Execution status | Evidence |
|---|---|---|
| Server unit tests | PASS | `npm run test:server` -> 414 pass / 0 fail / 5 skipped |
| API integration tests | PASS | `npm run test:api` -> 530 pass / 0 fail / 7 skipped |
| Frontend unit tests | PASS | `npm run test:frontend` -> 1309 tests passed |
| Security API cases | PASS | custom Node test group -> 104 pass / 0 fail / 2 skipped |
| HTTP header regression | PASS | `test/api/http.test.ts` -> 6/6 pass |
| E2E browser cases | BLOCKED / INCOMPLETE | `npm run cypress:run` aborted with exit code -1 |

## 4. Notable Observations

### Observed intentional challenge behavior

These were not classified as defects because they match the project’s design as an insecure CTF/security training application:

- SQL/UNION style search behavior,
- NoSQL challenge behavior in review/product flows,
- forged or manipulated JWT challenge behavior,
- user/password exposure in challenge-specific `whoami` queries,
- product tampering and basket access challenge flows.

### Requires review

Some observed behaviors were recorded as requires review rather than confirmed defects:

- malformed JSON login request producing a `500` HTML parser error page,
- anonymous `GET /profile` returning a `500` blocked-activity page,
- absence of CSP, HSTS, and Referrer-Policy from the local HTTP response,
- wildcard CORS configuration (`Access-Control-Allow-Origin: *`).

### Environment issue

- `redirect.spec.ts` failed on an external `owasp.org` page with React error #418.
- Cypress run later aborted before full completion.

This was recorded as an external target/environment problem, not as a confirmed Juice Shop application defect.

## 5. Test Coverage by Functionality

| Functionality | Coverage status |
|---|---|
| Registration | API pass; E2E partial |
| Login | API/security pass; E2E partial |
| Logout | Frontend unit pass |
| Search | API pass; E2E partial |
| Product | API pass; frontend pass |
| Basket | API pass; frontend pass |
| Checkout | API pass; E2E incomplete |
| Profile | API pass; E2E partial |
| Authentication | Server/API/security pass |
| Authorization | API/server/frontend pass |
| Response headers | PASS via dedicated HTTP suite |

## 6. Final Interpretation

The consolidated case set shows that the project’s automated and targeted security tests executed successfully, while the browser automation layer remains incomplete in this environment. The final classification should therefore be described as:

- PASS for completed suites,
- BLOCKED/INCOMPLETE for full E2E completion,
- OBSERVED/REQUIRES REVIEW for intentional challenge behavior and non-contractual error handling.

## 7. Evidence Files

- [AUTOMATED_TESTING.md](AUTOMATED_TESTING.md)
- [WHITEBOX_TESTING.md](WHITEBOX_TESTING.md)
- [ADVANCED_TESTING.md](ADVANCED_TESTING.md)
- [REGRESSION_TESTING.md](REGRESSION_TESTING.md)
