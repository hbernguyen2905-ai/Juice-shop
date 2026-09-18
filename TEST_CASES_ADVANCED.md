# Advanced Test Cases

| ID | Category | Target | Technique | Input / Action | Expected | Actual | Status | Classification |
|---|---|---|---|---|---|---|---|---|
| ADV-001 | Authentication | `/rest/user/login` | Negative testing | Missing email/password | Authentication rejected according to API contract | Existing API test passed with `401` | PASS | Normal behavior |
| ADV-002 | Authentication | `/rest/user/login` | Equivalence partitioning | Wrong email/password | Authentication rejected | Existing API test passed with `401` | PASS | Normal behavior |
| ADV-003 | JWT | `/rest/user/whoami` | Invalid token | `Authorization: Bearer invalid-token` | No authenticated user returned | `200`, `{"user":{}}` | PASS | Normal behavior |
| ADV-004 | JWT | `/rest/user/whoami` | Missing token | No authorization/cookie | No authenticated user returned | `200`, `{"user":{}}` | PASS | Normal behavior |
| ADV-005 | JWT | `/rest/user/whoami` | Expired token | Existing expired JWT fixture | No authenticated user returned | Existing API test passed with empty user | PASS | Normal behavior |
| ADV-006 | Authorization | `/rest/order-history/orders` | Role matrix | Customer/admin/accounting users | Only authorized role proceeds | Existing tests: customer/admin denied, accounting allowed | PASS | Normal behavior |
| ADV-007 | Authorization | `/rest/order-history/:id/delivery-status` | Role matrix | Customer/admin/accounting users | Only accounting role proceeds | Existing tests matched role boundary | PASS | Normal behavior |
| ADV-008 | Authorization | `/api/Cards` | Missing authentication | GET/POST without token | Protected API rejected | Existing payment tests returned `401` | PASS | Normal behavior |
| ADV-009 | SQL injection | `/rest/products/search` | Injection behavior | Quote/UNION payloads | Behavior classified against Juice Shop challenge contract | Existing tests confirm SQL errors and UNION extraction | OBSERVED | Existing intentional vulnerability |
| ADV-010 | SQL injection | `/rest/user/login` | Injection behavior | WHERE/UNION login payloads | Behavior classified against challenge contract | Existing tests confirm challenge login behavior | OBSERVED | Existing intentional vulnerability |
| ADV-011 | NoSQL injection | `/rest/products/:id/reviews` | Injection behavior | NoSQL-style review ID payload | Behavior classified against challenge contract | Existing API tests cover MongoDB sleep/injection behavior | OBSERVED | Existing intentional vulnerability |
| ADV-012 | XSS/sanitization | User/product/profile input | Stored/reflected input testing | HTML/script strings | Result classified by challenge mode | Existing tests cover XSS and safe-mode behavior | OBSERVED | Existing intentional vulnerability |
| ADV-013 | File upload | `/profile/image/file` | Invalid type | DOCX/non-image file | Reject unsupported content | Existing test returned `415` | PASS | Normal behavior |
| ADV-014 | File upload | `/profile/image/file` | Content validation | Plain text named as image/binary | Reject unrecognizable content | Existing test returned `500` Illegal file type | PASS | Normal behavior |
| ADV-015 | File upload | `/profile/image/file` | Authentication boundary | Valid image without auth | Reject anonymous upload | Existing test returned `500` blocked activity | PASS | Normal behavior |
| ADV-016 | File upload | `/file-upload` | Missing input | No multipart file | Return validation error | Local request returned `400` | PASS | Normal behavior |
| ADV-017 | Boundary | Search | Defined boundary | Query length behavior | Route limits query at 200 characters | Source/test evidence identifies 200-character cap | OBSERVED | Normal behavior |
| ADV-018 | Boundary | Payment card | Validation boundary | Invalid card number, month `13`, past year `2015` | Reject invalid card data | Existing tests returned `400` | PASS | Normal behavior |
| ADV-019 | Boundary | Basket | Numeric edge values | Zero, negative and very large quantity | Behavior recorded without load testing | Existing basket tests execute these challenge/validation cases | OBSERVED | Existing intentional vulnerability |
| ADV-020 | Error handling | `/rest/products/search` | Malformed input | `q=';` | Error response recorded | Local response `500` HTML SQL error | OBSERVED | Existing intentional vulnerability |
| ADV-021 | Error handling | `/rest/user/login` | Malformed JSON | Body `{` with JSON content type | Error response recorded | Local response `500` HTML SyntaxError page | OBSERVED | Requires review |
| ADV-022 | Error handling | `/profile` | Missing authentication | Anonymous GET | Contract-specific denial | Local response `500` HTML blocked-activity page; existing API test covers denial | OBSERVED | Requires review |
| ADV-023 | Security headers | `/` | Header inspection | Local GET | Record configured headers | `X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`, CORS `*`, Feature-Policy present | PASS | Normal behavior |
| ADV-024 | Security headers | `/` | Header absence inspection | Local GET | Record absent headers without assuming defect | No CSP, Referrer-Policy or HSTS observed | OBSERVED | Requires review |
| ADV-025 | Rate limiting | Password reset/2FA | Configuration review | Inspect middleware and existing tests | Identify safe testability | Rate limit configured at 100 requests/5 minutes; no dedicated rate-limit test found | NOT EXECUTED | Environment limitation |
| ADV-026 | Password security | User model/login/reset | Source + API behavior | Login, change/reset password | Password behavior matches existing contract | Existing tests cover hashing, login, change/reset and 2FA | PASS | Normal behavior |
| ADV-027 | Sensitive data | `/api/Cards` | Response inspection | Authorized card retrieval | Full card number should not be returned by card endpoint | Existing tests observe masked `************4321` | PASS | Normal behavior |
| ADV-028 | Sensitive data | `/rest/user/whoami` | Field exposure test | `fields=id,email,password` | Record behavior against challenge contract | Existing test confirms password can be returned | OBSERVED | Existing intentional vulnerability |
| ADV-029 | Security headers | HTTP test suite | Automated assertions | `test/api/http.test.ts` | Header assertions execute | Rerun after stopping app: `6 pass`, `0 fail` | PASS | Normal behavior |
| ADV-030 | Advanced automation | Security API group | Automated regression | 8 existing API spec files | Security group executes | `106 tests`, `104 pass`, `0 fail`, `2 skipped` | PASS | Normal behavior |

## Status notes

- `PASS` means the executed assertion or expected contract matched; it does not mean the entire application is secure.
- `OBSERVED` records behavior that was deliberately observed and classified, including intentional challenge behavior.
- `NOT EXECUTED` means no request burst was sent because no dedicated safe test was needed.
- No destructive, high-volume or external-system payload was used.