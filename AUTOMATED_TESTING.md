# Automated Testing

## 1. Objective

Đánh giá khả năng tự động hóa kiểm thử hiện có của OWASP Juice Shop qua server unit, API integration, frontend unit, Cypress E2E, coverage reporting, repeatability và CI configuration.

## 2. Automated Testing Infrastructure

| Layer | Framework/tool | Test location | Command | Coverage |
|---|---|---|---|---|
| Server Unit | Node.js built-in test runner, `tsx`, `assert` | `test/server/**/*.unit.test.ts` | `npm run test:server` | `nyc`, `npm run test:server:coverage` |
| API Integration | Node.js test runner, Supertest, `tsx` | `test/api/**/*.test.ts` | `npm run test:api` | `nyc`, `npm run test:api:coverage` |
| Frontend Unit | Angular unit-test builder, Vitest, Angular TestBed, jsdom | `frontend/src/**/*.spec.ts` | `npm run test:frontend` | Vitest V8, `npm run test:frontend:coverage` |
| E2E | Cypress 15.21.1, Electron 138 | `test/cypress/e2e/**.spec.ts` | `npm run cypress:run` / `npm run test:e2e` | No lcov configured |

Relevant configuration exists in [package.json](package.json), [frontend/package.json](frontend/package.json), [cypress.config.ts](cypress.config.ts) and [frontend/angular.json](frontend/angular.json).

## 3. Server Unit Automation

Command:

```text
npm run test:server
```

Final execution result:

- Tests: `419`
- Pass: `414`
- Fail: `0`
- Skipped: `5`
- Duration: `5975.694ms` on the repeatability run.

The suite covers security utilities, challenge logic, route handlers, startup helpers, file handling, order logic, XML/webhook behavior and other backend modules.

## 4. API Integration Automation

Command:

```text
npm run test:api
```

The first Phase 4 run produced `537 tests`, `529 pass`, `1 fail`, `7 skipped`, exit code `1`. The failing result did not reproduce on the immediate rerun.

Rerun result:

- Tests: `537`
- Pass: `530`
- Fail: `0`
- Skipped: `7`
- Duration: `43976.1458ms` on the rerun.

The API suite covers:

- Registration and user management.
- Login, authentication details, `whoami`, password change/reset and 2FA.
- Products, product search and product reviews.
- Basket items, basket access, coupons and checkout/order behavior.
- Cards, wallet and payment-related endpoints.
- Profile and profile image upload.
- Order history and role-based authorization.
- File serving, uploads, security challenges, WebSocket and B2B endpoints.

API coverage command:

```text
npm run test:api:coverage
```

Result:

- Tests: `537`
- Pass: `530`
- Fail: `0`
- Skipped: `7`
- Statements: `91.36% (3437/3762)`
- Branches: `76.92% (1570/2041)`
- Functions: `92.28% (670/726)`
- Lines: `91.71% (3265/3560)`

## 5. Frontend Unit Automation

Command:

```text
npm run test:frontend
```

Result:

- Test files: `121 passed`
- Tests: `1309 passed`
- Failed: `0`
- Duration: `92.07s`

Frontend coverage command:

```text
npm run test:frontend:coverage
```

Result:

- Test files: `121 passed`
- Tests: `1309 passed`
- Failed: `0`
- Coverage report: `frontend/coverage/frontend/index.html`
- lcov report: `frontend/coverage/frontend/lcov.info`
- Statements: `93.37% (8347/8939)`
- Branches: `90.5% (1955/2160)`
- Functions: `84.01% (1198/1426)`
- Lines: `96.08% (6162/6413)`

The frontend suite covers Angular components, services, guards, routing-related behavior, basket/payment/profile UI logic, forms and search-related components.

## 6. E2E Automation

Configuration facts:

- Base URL: `http://localhost:3000`.
- Browser observed: Electron 138 headless.
- Cypress version observed: `15.21.1`.
- Spec pattern: `test/cypress/e2e/**.spec.ts`.
- Specs discovered: `29`.
- Support file: `test/cypress/support/e2e.ts`.
- Custom commands: `test/cypress/support/commands.ts`.
- Fixtures: disabled (`fixturesFolder: false`).
- Downloads: `test/cypress/downloads`.
- Run retries: `2`.

Application startup action:

```text
npm start
```

The local server returned HTTP `200` at `http://localhost:3000` before Cypress execution.

Cypress command:

```text
npm run cypress:run
```

Execution status: `BLOCKED`.

Observed cause:

```text
Timed out waiting for the browser to connect.
The browser never connected. Something is wrong. The tests cannot run. Aborting...
```

The command exited with code `-1`. Some initial specs ran before the browser connection failure; the run did not produce a trustworthy aggregate total. The E2E aggregate fields are therefore `NOT AVAILABLE`, not fabricated as pass/fail totals.

Observed early results included passing administration, B2B order, basket, change-password, chatbot, complaint, contact, data-erasure and data-export scenarios. Complaint and contact each showed one pending test. Later specs, including `deluxe.spec.ts`, were aborted with zero tests after the browser stopped connecting.

## 7. Critical User Flows

| Flow | Existing automation mapping | Current result |
|---|---|---|
| Registration | `test/api/user.test.ts`, `test/api/helpers/auth.ts`, `test/cypress/e2e/register.spec.ts` | API PASS; Cypress blocked in full run |
| Login | `test/api/login.test.ts`, `test/cypress/e2e/login.spec.ts` | API PASS on rerun; Cypress blocked before login spec completed |
| Search | `test/api/search.test.ts`, `test/cypress/e2e/search.spec.ts` | API PASS; Cypress aggregate blocked |
| Product | `test/api/product.test.ts`, frontend product specs | API/frontend PASS |
| Basket | `test/api/basket.test.ts`, `test/cypress/e2e/basket.spec.ts` | API PASS; basket Cypress spec observed passing |
| Checkout | `test/api/basket.test.ts`, basket/order-related Cypress specs | API PASS; full E2E aggregate blocked |

This mapping describes existing automation; it does not claim that every Cypress flow completed in this environment.

## 8. Test Automation Matrix

| Test Layer | Tool | Scope | Automated? | Command | Result |
|---|---|---|---|---|---|
| Server Unit | Node test runner + tsx | Backend logic/modules | Yes | `npm run test:server` | 414 pass, 5 skipped |
| API Integration | Supertest + Node test runner | REST/API behavior | Yes | `npm run test:api` | 530 pass, 7 skipped on rerun |
| Frontend Unit | Vitest + Angular TestBed | Components/services/forms | Yes | `npm run test:frontend` | 1309 pass |
| E2E | Cypress/Electron | Browser user flows | Yes, execution blocked | `npm run cypress:run` | 29 discovered; aggregate NOT AVAILABLE; browser blocked |

## 9. Repeatability

Selected suite: `npm run test:server`.

| Run | Tests | Pass | Fail | Skipped | Duration |
|---|---:|---:|---:|---:|---:|
| Phase 4 first | 419 | 414 | 0 | 5 | 8524.2799ms |
| Phase 4 repeat | 419 | 414 | 0 | 5 | 5975.694ms |

The selected suite produced consistent test counts and pass/fail/skip results across the two executed runs. This is evidence for these two runs only, not a claim of universal stability.

The API suite had one transient failure on its first Phase 4 execution and passed on immediate rerun (`530/530`), so API repeatability should be treated as requiring further observation.

## 10. Coverage Reporting

Reports verified after commands:

- `coverage/server-tests/lcov.info`: generated.
- `coverage/api-tests/lcov.info`: generated.
- `frontend/coverage/frontend/lcov.info`: generated.
- `frontend/coverage/frontend/index.html`: generated.
- Frontend HTML report includes per-directory/per-file reports.

Server/API use `nyc` with `lcov` and `text-summary`. Frontend uses Vitest V8 with `lcovonly` and HTML reporters. Cypress does not have a configured lcov coverage report in the repository.

## 11. CI Automation

The GitHub Actions workflow [ci.yml](.github/workflows/ci.yml) contains automated jobs for:

- Frontend tests with coverage across OS/Node matrices.
- Server tests with coverage across OS/Node matrices.
- API tests with coverage across OS/Node matrices.
- Coverage artifact upload and Coveralls publishing on the configured push path.
- Server tests across multiple custom configurations: `7ms`, `addo`, `bodgeit`, `ctf`, `fbctf`, `juicebox`, `mozilla`, `oss`, `quiet`, `tutorial`, `unsafe`.
- Cypress E2E on Ubuntu and macOS with Chrome, using `npm start` and wait-on localhost.
- Smoke, Docker and other CI checks.

The GitHub Actions workflow itself was inspected, not executed locally.

## 12. Regression Automation

| Regression area | Automated suite | Command | Current result |
|---|---|---|---|
| Authentication | API tests, server unit tests, Cypress login specs | `npm run test:api`, `npm run test:server`, `npm run cypress:run` | API/server pass; Cypress blocked |
| Product/Search | API tests, frontend unit tests, Cypress search spec | `npm run test:api`, `npm run test:frontend` | API/frontend pass; Cypress aggregate blocked |
| Basket | API tests, frontend basket tests, Cypress basket spec | `npm run test:api`, `npm run test:frontend` | API/frontend pass; basket Cypress observations passed |
| Checkout | API basket/order tests, frontend order/payment tests, Cypress basket flow | Suite commands above | API/frontend pass; full E2E blocked |
| Authorization | API role tests, server insecurity tests, frontend guards | `npm run test:api`, `npm run test:server`, `npm run test:frontend` | Pass in executed suites |
| Frontend UI | Angular/Vitest suite and Cypress | `npm run test:frontend`, `npm run cypress:run` | Unit pass; E2E blocked |

## 13. Test Execution Results

| Command | Result |
|---|---|
| `npm run test:server` | 419 tests, 414 pass, 0 fail, 5 skipped |
| `npm run test:server:coverage` | Pass; coverage generated |
| `npm run test:api` first run | 537 tests, 529 pass, 1 fail, 7 skipped |
| `npm run test:api` rerun | 537 tests, 530 pass, 0 fail, 7 skipped |
| `npm run test:api:coverage` | 537 tests, 530 pass, 0 fail, 7 skipped; coverage generated |
| `npm run test:frontend` | 121 files, 1309 pass, 0 fail |
| `npm run test:frontend:coverage` | 121 files, 1309 pass; HTML/lcov generated |
| `npm start` | Server started, HTTP 200 verified, later stopped after E2E |
| `npm run cypress:run` | 29 specs discovered; BLOCKED; exit code -1 |

## 14. Limitations

- Cypress Electron stopped connecting partway through the 29-spec run; no aggregate E2E result is claimed.
- The first API run had one non-reproducible failure; the rerun passed. The underlying flaky test was not isolated further because the rerun completed cleanly and no source change was requested.
- CI workflow was inspected but not executed locally.
- Coverage metrics are suite/report metrics, not percentages of test cases.
- Existing intentional vulnerability challenge tests are included in API/E2E suites and should not be interpreted as production-security approval.

## 15. Conclusion

The repository has automated coverage at backend unit, API integration, frontend unit and E2E layers. Server, API rerun and frontend suites executed successfully with reports generated. Cypress configuration and specs are present, but full local E2E execution is blocked by browser connection failure in the current Windows environment.

## Evidence

- AT-E01: `package.json` and `frontend/package.json`, showing scripts/frameworks.
- AT-E02: `npm run test:server` output, showing 419/414/0/5.
- AT-E03: `npm run test:server:coverage` output and `coverage/server-tests/lcov.info`.
- AT-E04: `npm run test:api` rerun output, showing 537/530/0/7.
- AT-E05: `npm run test:api:coverage` output and `coverage/api-tests/lcov.info`.
- AT-E06: `npm run test:frontend` output, showing 121 files and 1309 tests passed.
- AT-E07: `npm run test:frontend:coverage` output and `frontend/coverage/frontend/index.html`.
- AT-E08: [cypress.config.ts](cypress.config.ts), showing base URL/spec/support configuration.
- AT-E09: `npm run cypress:run` output showing 29 specs discovered and browser connection block.
- AT-E10: Existing API/Cypress mappings in the Critical User Flows section.
- AT-E11: Verified lcov and HTML report paths in the Coverage Reporting section.
- AT-E12: [ci.yml](.github/workflows/ci.yml), showing CI test/coverage/E2E jobs.