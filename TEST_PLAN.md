# Test Plan

## 1. Overview

This test plan documents the validation strategy used for OWASP Juice Shop in the local Windows environment. The objective was to evaluate the real behavior of the application through black-box, white-box, automated, advanced, and regression checks without modifying production logic or pretending the application is secure by default.

OWASP Juice Shop is intentionally insecure and designed for security training. The testing therefore distinguishes between:

- confirmed defects introduced by a broken implementation,
- existing intentional challenge vulnerabilities,
- requires-review observations, and
- environment/external-target issues.

## 2. Scope

The test scope covered the application behavior as exercised through:

- server-side unit tests,
- API integration tests,
- frontend unit tests,
- advanced security-focused API tests,
- direct HTTP checks,
- local browser/E2E execution attempts.

In scope areas included:

- authentication and authorization,
- JWT handling,
- user/profile flows,
- basket/order/payment and checkout logic,
- product and search behavior,
- file upload and profile image handling,
- response headers and error handling,
- regression stability of existing suites.

Out of scope for this execution, unless explicitly required, were:

- destructive or high-volume testing,
- external penetration testing against third-party services,
- blanket brute-force or rate-limit stress testing,
- speculative vulnerability claims not supported by a real test run.

## 3. Objectives

1. Verify that existing automated suites still pass after prior test additions and analysis.
2. Reproduce and classify real execution results with evidence.
3. Distinguish intentional Juice Shop challenge vulnerabilities from new defects.
4. Record E2E/browser issues correctly as environment or target-side issues when evidence supports that classification.
5. Produce a final evidence-based report for academic use.

## 4. Test Environment

- OS: Windows
- Repository: OWASP Juice Shop 20.2.0
- Runtime: Node.js 22.x (project requirement)
- Server/API runner: Node.js built-in test runner + tsx
- Frontend runner: Angular + Vitest
- Browser automation: Cypress 15.x, Electron 138
- Base local URL used during browser checks: http://localhost:3000
- Source of truth for commands: [package.json](package.json)

## 5. Test Strategy

### 5.1 Black-box testing

Black-box validation focused on externally observable application behavior using HTTP/API requests, direct local checks, and existing route-level tests without inspecting internal implementation first.

### 5.2 White-box testing

White-box validation targeted selected backend security logic through direct module analysis and server unit tests, particularly:

- [lib/insecurity.ts](lib/insecurity.ts)
- [lib/utils.ts](lib/utils.ts)
- [test/server/insecurity.unit.test.ts](test/server/insecurity.unit.test.ts)
- [test/server/utils.unit.test.ts](test/server/utils.unit.test.ts)

The objective was to cover previously unhit branches without changing production code.

### 5.3 Automated testing

Existing project scripts were used as the source of truth for automated validation:

- `npm run lint`
- `npm run test:server`
- `npm run test:api`
- `npm run test:frontend`
- `npm run test:server:coverage`
- `npm run test:api:coverage`
- `npm run test:frontend:coverage`

### 5.4 Advanced testing

Advanced testing targeted risk-based security areas, including:

- authentication and authorization,
- JWT and token handling,
- file upload validation,
- input validation and injection behavior,
- error handling and response leakage,
- header inspection,
- rate-limit configuration analysis.

### 5.5 Regression testing

Regression validation checked whether recent Phase 3-5 testing work created any unintended breakage. It reused the actual project test suites and recorded final status with explicit evidence. No application source logic was changed during the Phase 6 regression run.

## 6. Test Cases and Evidence

The test-case inventories for the project are maintained separately in:

- [TEST_CASES_BLACKBOX.md](TEST_CASES_BLACKBOX.md)
- [TEST_CASES_WHITEBOX.md](TEST_CASES_WHITEBOX.md)
- [TEST_CASES_AUTOMATED.md](TEST_CASES_AUTOMATED.md)
- [TEST_CASES_ADVANCED.md](TEST_CASES_ADVANCED.md)

The consolidated final evidence used in this report is drawn from the executed runs and direct observations listed in:

- [AUTOMATED_TESTING.md](AUTOMATED_TESTING.md)
- [WHITEBOX_TESTING.md](WHITEBOX_TESTING.md)
- [ADVANCED_TESTING.md](ADVANCED_TESTING.md)
- [REGRESSION_TESTING.md](REGRESSION_TESTING.md)

## 7. Execution Summary

The actual execution evidence used for the final status is:

| Layer | Command | Status |
|---|---|---|
| Lint | `npm run lint` | PASS |
| Server unit | `npm run test:server` | PASS (414/419, 0 fail, 5 skipped) |
| API integration | `npm run test:api` | PASS (530/537, 0 fail, 7 skipped) |
| Frontend unit | `npm run test:frontend` | PASS (1309 tests passed) |
| Security API group | custom Node test invocation | PASS (104/106, 0 fail, 2 skipped) |
| HTTP headers | `test/api/http.test.ts` | PASS (6/6) |
| Cypress E2E | `npm run cypress:run` | BLOCKED / INCOMPLETE |

The E2E run did not reach a trustworthy full-suite aggregate because one redirect test failed on an external target and the run later exited with code -1.

## 8. Risks and Limitations

- The application intentionally exposes vulnerability challenge behavior; this is not a defect to be fixed as part of normal QA.
- Full E2E execution was blocked by a browser/target issue in the current environment.
- Some responses (for example malformed JSON and anonymous profile access) are observed as `500` pages, but they are recorded as requires review unless a product contract defines a different behavior.
- Rate-limit threshold verification was not executed because the project did not require a live abuse burst in this phase.

## 9. Acceptance Criteria

This test plan is considered satisfied when all of the following are true:

1. The automated suites that actually ran complete with pass status and recorded evidence.
2. The project’s intentional vulnerabilities are clearly separated from new defects.
3. E2E blockage is reported as environment or external-target issue, not as a false application failure.
4. Final reporting is based on the latest actual execution output, not earlier baseline numbers.

## 10. Final Assessment

The project is suitable for evidence-based validation in a training/security context. The executed backend, API, frontend, and targeted security suites passed. The full Cypress run was incomplete and therefore cannot be treated as a project-wide pass. The final report must reflect that distinction explicitly.
