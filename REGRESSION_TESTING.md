# Regression Testing

## 1. Objective

Xác nhận bằng test execution thực tế rằng các thay đổi của các phase trước, đặc biệt bốn test bổ sung trong Phase 3 và các báo cáo Phase 3-5, không gây regression cho server, API, frontend và các security regression suites hiện có.

Không có application source code, business logic hoặc dependency nào được sửa trong Phase 6.

## 2. Test Environment

- OS: Windows.
- Node.js: `v22.22.3`.
- Repository: OWASP Juice Shop `20.2.0`.
- Server/API unit runner: Node.js built-in test runner + `tsx`.
- API client: Supertest.
- Frontend: Angular unit-test builder + Vitest/jsdom.
- E2E: Cypress `15.21.1`, Electron `138`.
- E2E base URL: `http://localhost:3000`.
- API/server tests use the repository test environment and in-memory database setup.

## 3. Test Suite

| Suite | Command | Scope |
|---|---|---|
| Lint | `npm run lint` | Root/backend/test lint, config validation, frontend TS/SCSS lint |
| Server unit | `npm run test:server` | `test/server/**/*.unit.test.ts` |
| Server coverage | `npm run test:server:coverage` | Server suite + nyc reports |
| API integration | `npm run test:api` | `test/api/**/*.test.ts` |
| API coverage | `npm run test:api:coverage` | API suite + nyc reports |
| Frontend unit | `npm run test:frontend` | `frontend/src/**/*.spec.ts` |
| Frontend coverage | `npm run test:frontend:coverage` | Frontend suite + Vitest V8 reports |
| Security regression | Explicit Node test command for 8 existing security API files | Authentication, authorization, JWT, injection, upload, profile |
| HTTP header regression | Explicit Node test command for `test/api/http.test.ts` | CORS, frameguard, nosniff and related headers |
| E2E | `npm run cypress:run` | 29 Cypress specs discovered |

## 4. Test Execution

### Initial Git state

`git status --short` before execution showed:

Modified:

- `test/server/insecurity.unit.test.ts`
- `test/server/utils.unit.test.ts`

Untracked reports:

- `ADVANCED_TESTING.md`
- `AUTOMATED_TESTING.md`
- `TEST_CASES_ADVANCED.md`
- `TEST_CASES_AUTOMATED.md`
- `TEST_CASES_BLACKBOX.md`
- `TEST_CASES_WHITEBOX.md`
- `WHITEBOX_TESTING.md`

The relevant diff was limited to the four Phase 3 white-box tests in the two server test files. No reset, checkout or revert was performed.

### Lint

Command:

```text
npm run lint
```

Result: `PASS`.

The command completed root ESLint, config schema validation, Angular lint and SCSS stylelint successfully.

### Server unit

Command:

```text
npm run test:server
```

Phase 6 result:

- Total: `419`
- PASS: `414`
- FAIL: `0`
- SKIPPED: `5`
- Duration: `13264.3896ms`

### Frontend unit

Command:

```text
npm run test:frontend
```

Phase 6 result:

- Test files: `121 passed`
- Tests: `1309 passed`
- FAIL: `0`
- Duration: `70.91s`
- SKIPPED: not reported by this output

### API integration

Command:

```text
npm run test:api
```

Phase 6 result:

- Total: `537`
- PASS: `530`
- FAIL: `0`
- SKIPPED: `7`
- Duration: `36830.0189ms`

No rerun was required because the first Phase 6 execution passed.

### Security regression

Command:

```text
node --import ./test/api/helpers/test-env.mjs --import tsx --test --test-force-exit test/api/login.test.ts test/api/user.test.ts test/api/authenticated-users.test.ts test/api/2fa.test.ts test/api/order-history.test.ts test/api/search.test.ts test/api/product.test.ts test/api/profile-image-upload.test.ts
```

Result:

- Total: `106`
- PASS: `104`
- FAIL: `0`
- SKIPPED: `2`
- Duration: `6143.7038ms`

This suite covers authentication, authorization, JWT/2FA, search injection behavior, product behavior, profile upload, password-related flows and sensitive-data assertions.

### HTTP/security header regression

Command:

```text
node --import ./test/api/helpers/test-env.mjs --import tsx --test --test-force-exit test/api/http.test.ts
```

Result:

- Total: `6`
- PASS: `6`
- FAIL: `0`
- SKIPPED: `0`
- Duration: `3421.9922ms`

The same test had previously encountered an environment precondition while the application server was running. It passed when rerun in the clean test process without the external server.

### E2E

Application startup:

```text
npm start
```

Local readiness check returned HTTP `200` from `http://localhost:3000`.

Command:

```text
npm run cypress:run
```

Observed result:

- Specs discovered: `29`.
- The run progressed beyond the previous Phase 4 browser failure.
- `redirect.spec.ts`: `3 tests`, `2 passing`, `1 failing`.
- `profile.spec.ts`: `5 tests`, `3 passing`, `2 pending`.
- `restApi.spec.ts`: `4 tests`, `3 passing`, `1 pending`.
- `search.spec.ts`: `7 tests`, `4 passing`, `3 pending`.
- The process stopped at `totpSetup.spec.ts` and exited with code `-1`.
- No aggregate all-spec result was produced.

The failing redirect test produced an uncaught React error #418 from the external `owasp.org` page after the redirect target was reached. This is classified as an external target/environment issue, not a confirmed Juice Shop application defect. Because the run did not complete, E2E status is `BLOCKED / INCOMPLETE`, not PASS.

The local server was stopped after the E2E attempt.

## 5. PASS/FAIL/SKIP

| Execution | Total | PASS | FAIL | SKIPPED | Status |
|---|---:|---:|---:|---:|---|
| Lint | N/A | PASS | 0 | N/A | PASS |
| Server unit | 419 | 414 | 0 | 5 | PASS |
| Frontend unit | 1309 tests / 121 files | 1309 | 0 | Not reported | PASS |
| API integration | 537 | 530 | 0 | 7 | PASS |
| Security API group | 106 | 104 | 0 | 2 | PASS |
| HTTP headers | 6 | 6 | 0 | 0 | PASS |
| Cypress E2E | 29 specs discovered | Aggregate N/A | 1 observed before abort | Pending/aggregate N/A | BLOCKED / INCOMPLETE |

## 6. Regression Findings

### Critical user flows

| Function | Regression evidence | Result |
|---|---|---|
| Register | API suite includes `test/api/user.test.ts`; Cypress reached register spec before E2E abort and its observed result was 4 passing | PASS for API; E2E partial |
| Login | API login tests and security group passed; Cypress `login.spec.ts` observed 15 passing | PASS for API/security; E2E partial |
| Logout | Frontend unit suite includes navbar/sidenav logout tests and passed; no separate Phase 6 manual logout run | PASS via frontend unit; not directly browser-tested |
| Search | API suite/search tests and security group passed; Cypress search spec observed 4 passing and 3 pending | PASS for API; E2E partial |
| Product | API product tests and frontend product tests passed | PASS |
| Cart/Basket | API basket tests passed; Cypress basket spec was part of the executed run before abort | PASS for API; E2E aggregate incomplete |
| Checkout | API basket/order tests passed; no complete Cypress aggregate | PASS for API; E2E incomplete |
| User profile | API profile/upload tests passed; Cypress profile had 3 passing and 2 pending | PASS for API; E2E partial |
| Authentication | Server/API/security suites passed | PASS |
| Authorization | API order-history role tests and frontend guards passed | PASS |

No regression is concluded for a function solely because its source was unchanged; the classifications above are based on the executed tests listed.

## 7. Coverage

### Server final coverage

Command: `npm run test:server:coverage`

- Statements: `40.90% (1332/3256)`
- Branches: `45.68% (751/1644)`
- Functions: `39.28% (262/667)`
- Lines: `41.16% (1263/3068)`

Report: `coverage/server-tests/lcov.info`.

### API final coverage

Command: `npm run test:api:coverage`

- Statements: `91.36% (3437/3762)`
- Branches: `76.92% (1570/2041)`
- Functions: `92.28% (670/726)`
- Lines: `91.71% (3265/3560)`

Report: `coverage/api-tests/lcov.info`.

### Frontend final coverage

Command: `npm run test:frontend:coverage`

- Statements: `92.85% (8300/8939)`
- Branches: `90.13% (1947/2160)`
- Functions: `84.01% (1198/1426)`
- Lines: `95.38% (6117/6413)`

Reports:

- `frontend/coverage/frontend/lcov.info`
- `frontend/coverage/frontend/index.html`

Condition coverage was not provided by the available tools and is `NOT AVAILABLE`.

## 8. Defects

| ID | Test | Expected | Actual | Cause | Classification | Evidence |
|---|---|---|---|---|---|---|
| REG-001 | Cypress `redirect.spec.ts` allowlisted-target test | Redirect flow completes without browser-side uncaught error | External `owasp.org` page emitted React error #418; test failed after 3 attempts | External target/runtime behavior | Environment/external-target issue; requires review | Cypress log and screenshots in `cypress/screenshots/redirect.spec.ts/` |
| REG-002 | Remaining Cypress run at `totpSetup.spec.ts` | All 29 specs complete | Process exited `-1` before aggregate completion | E2E/browser execution environment | Environment/infrastructure issue | Cypress log; no aggregate result |

No confirmed Juice Shop application defect was established by Phase 6. Existing SQL, NoSQL, XSS, broken authorization, forged JWT and data-exposure challenge behavior remains intentional and is not listed as a new regression defect.

## 9. Final Testing Status

The server unit, API integration, frontend unit, security API and HTTP header regression suites passed with their reported skips. Lint passed. E2E executed further than Phase 4 but did not complete: one external-target failure was observed and the process later aborted with exit `-1`. Therefore the overall final status is:

```text
REGRESSION STATUS: PASS for completed automated suites; BLOCKED/INCOMPLETE for full E2E.
```

## Final Test Summary

The table avoids adding the same suite multiple times across phases.

| Testing Method | Tests | PASS | FAIL | SKIP | Notes |
|---|---:|---:|---:|---:|---|
| Black-box | N/A | N/A | N/A | N/A | Phase 6 reused API/HTTP automation; no separate black-box-only run was executed |
| White-box | 419 server tests | 414 | 0 | 5 | Server suite includes the Phase 3 white-box tests; not added to another total |
| Automated | 537 API + 1309 frontend tests | 1839 | 0 | 7 API | Aggregated only across distinct Phase 6 server/API/frontend execution layers; server is shown separately to avoid ambiguity |
| Advanced | 106 security API + 6 HTTP tests | 110 | 0 | 2 | Distinct targeted regression commands; these overlap with broader API suite and are not added to overall total |
| Regression | See individual suites | N/A | N/A | N/A | Regression is a status assessment, not a new independent test population |

## Remaining Issues

- Full Cypress aggregate remains unavailable because the run aborted with exit `-1`.
- One redirect E2E test failed due to an uncaught error from external `owasp.org` content.
- Five server and seven API skips remain as reported by the test runner.
- Rate-limit threshold runtime was not executed.
- Existing security header observations and malformed-input error observations remain requires-review items from Phase 5.
- Frontend final coverage differs from the Phase 4 report and is recorded from the current Phase 6 HTML report: `92.85%` statements, `90.13%` branches, `84.01%` functions, `95.38%` lines.

## Evidence Checklist

- REG-E01 — Git status/diff before testing.
- REG-E02 — `npm run lint` pass.
- REG-E03 — `npm run test:server` result.
- REG-E04 — `npm run test:server:coverage` result and lcov.
- REG-E05 — `npm run test:api` result.
- REG-E06 — `npm run test:api:coverage` result and lcov.
- REG-E07 — `npm run test:frontend` result.
- REG-E08 — `npm run test:frontend:coverage` result and HTML/lcov.
- REG-E09 — Cypress Phase 6 log, screenshots and exit `-1`.
- REG-E10 — Security API group and HTTP header regression outputs.
- REG-E11 — Critical user-flow regression mapping in section 6.
- REG-E12 — Final server/API/frontend coverage metrics in section 7.
- REG-E13 — Final test summary in this report.
- REG-E14 — Defect analysis table in section 8.