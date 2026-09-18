# Final Test Report

## 1. Project Context

OWASP Juice Shop is an intentionally insecure web application created for security training. This final report evaluates the project using evidence gathered from actual local test execution, not by assuming the application is secure.

The project was assessed using a multi-layered strategy:

- black-box observation,
- white-box targeted security logic review,
- automated server/API/frontend test runs,
- advanced security-focused validation,
- regression execution and comparison,
- browser automation attempt for E2E coverage.

## 2. Objectives

1. Verify automated project suites still pass with current local environment configuration.
2. Distinguish intentional vulnerability challenge behavior from defects.
3. Document the real E2E limitation in this environment.
4. Produce a final academically usable report based on executed evidence.

## 3. Methods Used

### Automated and direct validation

The following commands were executed and recorded as the primary evidence source:

- `npm run lint`
- `npm run test:server`
- `npm run test:api`
- `npm run test:frontend`
- `npm run test:server:coverage`
- `npm run test:api:coverage`
- `npm run test:frontend:coverage`
- custom Node-based security API group
- `node --import ./test/api/helpers/test-env.mjs --import tsx --test --test-force-exit test/api/http.test.ts`
- `npm run cypress:run`

### White-box validation

The selected backend security modules were inspected and their branch paths tested through direct server unit additions in:

- [test/server/insecurity.unit.test.ts](test/server/insecurity.unit.test.ts)
- [test/server/utils.unit.test.ts](test/server/utils.unit.test.ts)

### Additional reports

The phase-level reports are:

- [AUTOMATED_TESTING.md](AUTOMATED_TESTING.md)
- [WHITEBOX_TESTING.md](WHITEBOX_TESTING.md)
- [ADVANCED_TESTING.md](ADVANCED_TESTING.md)
- [REGRESSION_TESTING.md](REGRESSION_TESTING.md)

## 4. Actual Execution Results

### 4.1 Lint

Status: PASS

Evidence:

- `npm run lint` completed successfully.

### 4.2 Server unit tests

Status: PASS

Result:

- 419 total
- 414 pass
- 0 fail
- 5 skipped

### 4.3 API integration tests

Status: PASS

Result:

- 537 total
- 530 pass
- 0 fail
- 7 skipped

### 4.4 Frontend unit tests

Status: PASS

Result:

- 121 test files passed
- 1309 tests passed
- 0 failed

### 4.5 Security API subset

Status: PASS

Result:

- 106 total
- 104 pass
- 0 fail
- 2 skipped

### 4.6 HTTP security header suite

Status: PASS

Result:

- 6 total
- 6 pass
- 0 fail
- 0 skipped

### 4.7 E2E browser run

Status: BLOCKED / INCOMPLETE

Evidence:

- `npm run cypress:run` discovered 29 specs.
- One redirect test failed after the browser reached external `owasp.org` content with React error #418.
- The run later exited with code -1 before aggregate completion.

This was classified as an environment/external-target issue, not a confirmed Juice Shop application defect.

## 5. Coverage Summary

Final actual coverage values recorded from the executed runs:

| Layer | Statements | Branches | Functions | Lines |
|---|---:|---:|---:|---:|
| Server | 40.90% | 45.68% | 39.28% | 41.16% |
| API | 91.36% | 76.92% | 92.28% | 91.71% |
| Frontend | 92.85% | 90.13% | 84.01% | 95.38% |

Condition coverage was not provided by the available coverage tooling in this execution and is recorded as NOT AVAILABLE.

## 6. Defect Classification

### Confirmed defects

- None established in the executed suites.

### Existing vulnerabilities

The project intentionally contains challenge behavior such as:

- SQL/UNION injection patterns,
- NoSQL-related insecure flows,
- tampering and broken authorization scenarios,
- JWT forgery or token misuse flows,
- password and data exposure challenge behavior,
- XSS/SSTI challenge paths.

These are intentional and should not be reported as new defects.

### Requires review

Observed but not definitive defects without a product requirement or contract:

- malformed JSON login `500` HTML parser page,
- anonymous `GET /profile` blocked-activity HTML error,
- missing CSP/HSTS/Referrer-Policy in local HTTP responses,
- wildcard CORS configuration.

### Environment issues

- external browser redirect target produced an uncaught React error,
- Cypress aborted before full suite completion.

## 7. Overall Status

The final status is:

```text
PASS for all completed automated and targeted security suites.
BLOCKED / INCOMPLETE for the full Cypress E2E execution in this environment.
```

This is the correct classification for the current evidence and should be preserved in the academic report.

## 8. Limitations

- Full browser automation could not complete due to environment/external-target failure.
- Rate-limit threshold testing was not executed.
- No destructive or large-scale abuse testing was performed.
- Observed `500` HTML error pages were recorded as requires review rather than confirmed defects.
- The application is intentionally insecure; this is part of the project’s purpose.

## 9. Final Conclusion

The project demonstrates strong automated backend/API/frontend validation and targeted security coverage. The completed suites passed with real output. The main unresolved issue is the incomplete E2E run caused by external browser conditions, and the project’s intentional vulnerabilities must remain clearly distinguished from genuine defects. The final documentation should therefore present the application as a deliberately insecure training project with strong automated validation, not as a secure production system.

## 10. Evidence Core Files

- [REGRESSION_TESTING.md](REGRESSION_TESTING.md)
- [AUTOMATED_TESTING.md](AUTOMATED_TESTING.md)
- [WHITEBOX_TESTING.md](WHITEBOX_TESTING.md)
- [ADVANCED_TESTING.md](ADVANCED_TESTING.md)
- [COVERAGE_REPORT.md](COVERAGE_REPORT.md)
- [DEFECTS.md](DEFECTS.md)

PHASE 7 COMPLETED — TESTING PROJECT REPORT READY
