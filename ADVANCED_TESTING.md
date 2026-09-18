# Advanced Testing

## 1. Objective

Thực hiện kiểm thử nâng cao theo risk đối với các khu vực security-sensitive của Juice Shop trong local environment, sử dụng source/test infrastructure hiện có và không thay đổi business logic.

## 2. Scope

Authentication, authorization, JWT, input validation, SQL/NoSQL injection behavior, XSS/sanitization behavior, file upload, boundaries, error handling, security headers, rate limiting configuration, password behavior và sensitive-data responses.

## 3. Risk-based Test Strategy

| Area | Component/File or endpoint | Risk | Existing test evidence | Advanced test needed/result |
|---|---|---|---|---|
| Authentication | `routes/login.ts`, `/rest/user/login` | Credential/session handling | `test/api/login.test.ts` | Executed; representative negative/auth tests pass |
| Authorization | `lib/insecurity.ts`, order/card/basket APIs | Role boundary/IDOR | `test/api/order-history.test.ts`, basket/payment tests | Executed; role matrix pass; challenge behavior observed where applicable |
| JWT | `lib/insecurity.ts`, `utils.jwtFrom` | Invalid/expired/malformed token | `test/api/user.test.ts`, `2fa.test.ts`, server insecurity tests | Executed; invalid/missing/expired cases pass |
| Search/Input | `routes/search.ts` | SQL metacharacters and length | `test/api/search.test.ts` | Executed; SQL challenge behavior observed |
| SQL/NoSQL | Search/login/review endpoints | Injection/data exposure | search/login/product-review tests | Executed; classified as existing intentional challenge behavior |
| XSS | User/product/profile inputs | HTML/script persistence/rendering | user/product/profile tests | Executed through existing tests; challenge behavior observed |
| File upload | `routes/fileUpload.ts`, profile upload | MIME, content, filename and size | `profile-image-upload.test.ts`, Cypress complain tests | Executed representative API tests; missing-file local check pass |
| Payment | Card/wallet routes | Card validation/exposure | `payment.test.ts`, wallet tests | Executed through existing API suite; masking pass |
| Password | User model, login/change/reset | Hashing and password flow | login/password/2FA tests | Executed through advanced API group |
| Rate limit | `server.ts` reset-password/2FA middleware | Abuse protection | No dedicated rate-limit test found | Configuration inspected; request burst not executed |
| Security headers | `server.ts`, `test/api/http.test.ts` | Browser/security policy headers | `http.test.ts` | Executed; configured headers observed |
| Error handling | `server.ts`, route error handlers | Status/error leakage | `api.test.ts`, route tests | Executed negative requests; some intentional/known error disclosure observed |

## 4. Authentication Testing

Command:

```text
node --import ./test/api/helpers/test-env.mjs --import tsx --test --test-force-exit test/api/login.test.ts test/api/user.test.ts test/api/authenticated-users.test.ts test/api/2fa.test.ts test/api/order-history.test.ts test/api/search.test.ts test/api/product.test.ts test/api/profile-image-upload.test.ts
```

Result: `106 tests`, `104 pass`, `0 fail`, `2 skipped`.

Covered missing credentials, invalid credentials, 2FA, invalid/expired tokens, authentication details, password-related flows and protected resources.

Direct local checks:

- Missing token on `GET /rest/user/whoami`: HTTP `200`, body `{"user":{}}`.
- Invalid bearer token on the same endpoint: HTTP `200`, body `{"user":{}}`.
- Login with valid admin credentials: HTTP `200`, authentication token returned.

These `200` empty-user responses are the existing application contract tested by the repository; they are not reclassified as failures.

## 5. Authorization Testing

Role boundary evidence from existing API tests:

- Customer/admin cannot retrieve all accounting orders.
- Accounting can retrieve all orders.
- Customer/admin cannot change delivery status.
- Accounting can change delivery status.
- Unauthenticated card access is rejected with `401`.
- Protected basket/profile paths require authentication according to their existing tests.

The product tampering, basket access, forged JWT and related cases are classified as `Existing intentional vulnerability` where the test explicitly targets a Juice Shop challenge.

## 6. JWT Testing

Tested via existing API/server infrastructure:

- Missing token.
- Invalid token.
- Broken authorization format.
- Expired token.
- 2FA temporary token and invalid TOTP cases.
- Role claims and authenticated-user mapping.

No new JWT implementation or signing helper was introduced.

## 7. Input Validation

Representative invalid inputs were exercised through existing tests and local requests:

- Empty/missing login credentials.
- Invalid user role.
- Invalid card number/month/year.
- Invalid product and basket IDs.
- Missing upload file.
- Invalid file content/type.
- Malformed JSON.
- Search SQL metacharacters.

No oversized or high-volume payload was used.

## 8. Injection Testing

Existing search/login/review API tests exercise SQL and NoSQL-style payloads. Results include SQL errors, UNION extraction and review injection behavior intentionally present in Juice Shop challenges.

Classification:

```text
OBSERVED — Existing intentional vulnerability/challenge behavior
```

No new injection defect is claimed.

## 9. XSS/Sanitization

Existing server/API tests cover:

- HTML/script input in user/product data.
- Sanitization helper behavior.
- Profile/SSTI challenge paths.
- Safe-mode behavior.

The challenge-specific unsafe behavior is recorded as intentional challenge behavior. No browser script execution or persistent external payload was introduced.

## 10. File Upload Testing

Source evidence in `routes/fileUpload.ts` and profile upload routes shows:

- File presence validation.
- File size challenge threshold at `100000` bytes and multer memory limit `200000` bytes.
- Extension checks for PDF/XML/ZIP/YML/YAML in complaint upload processing.
- Binary content inspection for profile images.
- Filename sanitization in server upload storage.

Executed representative results:

- Valid JPG profile image: existing test passes with redirect.
- Unsupported DOCX/non-image: `415`.
- Unrecognizable content: `500` Illegal file type.
- Anonymous profile image upload: blocked by existing route behavior.
- Missing `/file-upload` file: local response `400`.

No executable, malware or destructive archive was uploaded.

## 11. Boundary Testing

Boundaries supported by source/tests:

- Search query cap: `200` characters.
- Card expiration month: `13` rejected.
- Past card year: `2015` rejected in existing test data.
- Card number invalid format: rejected.
- Upload challenge size threshold: `100000` bytes; multer limit: `200000` bytes.
- Basket quantity zero/negative/large cases are present in existing tests.

For unspecified email/password maximums, no invented boundary was used: `Boundary not explicitly defined`.

## 12. Error Handling

Small local negative checks produced:

| Request | Actual response | Classification |
|---|---|---|
| `GET /rest/user/whoami` without token | `200`, empty user | Normal behavior |
| Same endpoint with invalid token | `200`, empty user | Normal behavior |
| Search `q=';` | `500` HTML SQL error | Existing intentional vulnerability |
| Anonymous `GET /profile` | `500` blocked-activity HTML error page | Requires review; existing route/test behavior |
| `POST /file-upload` without file | `400` | Normal validation behavior |
| Login with malformed JSON `{` | `500` HTML SyntaxError page | Requires review; observed error handling |

The last two `500` responses are observations, not confirmed defects, because no separate product error contract was supplied.

## 13. Security Headers

Direct local `GET /` returned:

- `X-Frame-Options: SAMEORIGIN`.
- `X-Content-Type-Options: nosniff`.
- `Access-Control-Allow-Origin: *`.
- `Feature-Policy: payment 'self'`.
- `X-Recruiting: /#/jobs`.

Not observed in the response:

- `Content-Security-Policy`.
- `Referrer-Policy`.
- `Strict-Transport-Security`.

The repository's `test/api/http.test.ts` was rerun after stopping the app server and passed `6/6`. Header absence is recorded as `OBSERVED / Requires review`, not automatically a defect.

## 14. Rate Limiting

Source inspection confirmed `express-rate-limit` middleware on:

- `/rest/user/reset-password`: `100` requests per `5` minutes.
- `/rest/2fa/verify`: `100` requests per `5` minutes.
- `/rest/2fa/setup`: `100` requests per `5` minutes.
- `/rest/2fa/disable`: `100` requests per `5` minutes.

No dedicated rate-limit tests were found in `test/`. No request burst was sent to trigger limits, to avoid unnecessary abuse traffic.

Status: `NOT EXECUTED` for runtime threshold verification.

## 15. Password Security

Source/test evidence covers:

- Password hashing before storage in the User model.
- Login password comparison.
- Password change and reset negative paths.
- 2FA setup/verification.
- User API responses excluding password in normal fields.

The repository intentionally includes password-related challenge behavior; this was not modified or interpreted as a new defect.

## 16. Sensitive Data Exposure

Observed/tested:

- Payment card responses are masked, e.g. `************4321`.
- Normal user listing excludes password.
- Authentication-details replaces password with asterisks.
- `whoami?fields=id,email,password` can return password data; existing test explicitly classifies this as a Juice Shop challenge behavior.
- Search UNION challenge can expose user/hash data; classified as intentional.
- Error pages can include SQL/parser/internal messages; classified as observed and requires review where not explicitly challenge behavior.

## 17. Automated Advanced Tests

The representative advanced group used existing API test infrastructure:

- Files: login, user, authenticated-users, 2FA, order-history, search, product and profile-image-upload API specs.
- Result: `106 tests`, `104 pass`, `0 fail`, `2 skipped`.
- Header suite: `test/api/http.test.ts`, rerun result `6 pass`, `0 fail`, `0 skipped`.

No new test file or dependency was added because existing tests already exercise the selected advanced risks without duplication.

## 18. Findings

### Existing intentional vulnerabilities/challenge behavior

- SQL injection/UNION behavior in product search and login.
- NoSQL injection behavior in product reviews.
- XSS/SSTI-related challenge paths.
- Product tampering, basket access and forged JWT challenge paths.
- Password exposure through explicitly requested `whoami` fields.

### Potential issue / requires review

- Malformed JSON login request returns a `500` HTML SyntaxError page with parser details.
- Anonymous profile request returns a `500` HTML blocked-activity error page.
- Security response did not include CSP, Referrer-Policy or HSTS in the local HTTP response.
- CORS allows all origins (`*`), as intentionally configured and asserted by existing HTTP tests.

These are not confirmed defects without a product/security requirement that defines a different contract.

### Confirmed test failures

- None in the final advanced API group or rerun HTTP header suite.

## 19. Limitations

- Runtime rate-limit threshold was not triggered.
- No DoS, load, brute-force or large-payload testing was performed.
- No external targets were contacted for attack testing.
- Cypress was not used because Phase 4 established browser connection blocking in this environment.
- Advanced API results rely on existing tests and their challenge-aware expectations.
- Header absence is observation only, not proof of a security defect.

## 20. Evidence

- ADV-E01: Advanced API group execution, `106 tests`, `104 pass`, `0 fail`, `2 skipped`.
- ADV-E02: Existing authorization role tests for customer/admin/accounting boundaries.
- ADV-E03: Local negative checks for missing/invalid JWT, missing upload and malformed JSON.
- ADV-E04: Existing search/login/review injection test output.
- ADV-E05: Existing sanitization/profile/product XSS-related test output.
- ADV-E06: Profile/file upload tests and local missing-file check.
- ADV-E07: Source/test evidence for search, card, quantity and upload boundaries.
- ADV-E08: Local error response checks for search, profile and malformed JSON.
- ADV-E09: Direct local response headers and `test/api/http.test.ts` `6/6` pass.
- ADV-E10: Existing payment/user/authentication-details/whoami response assertions.
- ADV-E11: Automated advanced API group and HTTP header suite execution.
- ADV-E12: [TEST_CASES_ADVANCED.md](TEST_CASES_ADVANCED.md) and this report.

## 21. Conclusion

Advanced testing exercised representative high-risk areas using local API tests, existing security tests, source evidence and a small set of direct HTTP checks. The final advanced API group and HTTP header suite passed their assertions. Intentional Juice Shop vulnerabilities were recorded as observed challenge behavior, while malformed-input errors and absent headers were classified as requires review rather than new confirmed defects. Rate-limit threshold execution and destructive/high-volume testing were intentionally not performed.