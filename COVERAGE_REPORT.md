# Coverage Report

## 1. Purpose

This document records the final coverage values produced by the executed test suites in the local project environment. It is based on the actual `nyc` and Vitest coverage output captured during the validation run and should be read as evidence, not as a claim of full application security.

## 2. Coverage Commands Used

- Server: `npm run test:server:coverage`
- API: `npm run test:api:coverage`
- Frontend: `npm run test:frontend:coverage`

## 3. Server Coverage

Command run:

```text
npm run test:server:coverage
```

Final values:

- Statements: 40.90% (1332/3256)
- Branches: 45.68% (751/1644)
- Functions: 39.28% (262/667)
- Lines: 41.16% (1263/3068)

Report location:

- `coverage/server-tests/lcov.info`

## 4. API Coverage

Command run:

```text
npm run test:api:coverage
```

Final values:

- Statements: 91.36% (3437/3762)
- Branches: 76.92% (1570/2041)
- Functions: 92.28% (670/726)
- Lines: 91.71% (3265/3560)

Report location:

- `coverage/api-tests/lcov.info`

## 5. Frontend Coverage

Command run:

```text
npm run test:frontend:coverage
```

Final values:

- Statements: 92.85% (8300/8939)
- Branches: 90.13% (1947/2160)
- Functions: 84.01% (1198/1426)
- Lines: 95.38% (6117/6413)

Report locations:

- `frontend/coverage/frontend/lcov.info`
- `frontend/coverage/frontend/index.html`

## 6. Condition Coverage

Condition coverage was not provided by the available tooling output in this execution. It is therefore recorded as:

- Condition coverage: NOT AVAILABLE

## 7. Interpretation

The coverage figures are valid for the executed suite reports and are not intended to imply a security-assured application. OWASP Juice Shop intentionally includes challenge-grade vulnerabilities and hostile input scenarios, so the coverage numbers reflect test execution breadth and code reach, not security robustness.

## 8. Evidence Summary

Coverage values were recorded from the actual executed reports during the final Phase 6 validation run and are consistent with the final project documentation in:

- [REGRESSION_TESTING.md](REGRESSION_TESTING.md)
- [WHITEBOX_TESTING.md](WHITEBOX_TESTING.md)
- [AUTOMATED_TESTING.md](AUTOMATED_TESTING.md)
