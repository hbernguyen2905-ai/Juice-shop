# Automated Test Cases

| ID | Layer | Tool | Test/Flow | Command | Expected | Actual | Status |
|---|---|---|---|---|---|---|---|
| AT-001 | Server Unit | Node test runner | Backend unit regression | `npm run test:server` | Unit suite executes with assertions | 419 tests, 414 pass, 0 fail, 5 skipped | PASS |
| AT-002 | Server Coverage | nyc | Backend coverage report | `npm run test:server:coverage` | lcov/text-summary generated | Statements 40.90%, branches 45.68%, functions 39.28%, lines 41.16% | PASS |
| AT-003 | API Integration | Supertest | Authentication and registration API regression | `npm run test:api` | API suite executes | 537 tests, 530 pass, 0 fail, 7 skipped on rerun | PASS |
| AT-004 | API Integration | Supertest | Search/product/basket/checkout regression | `npm run test:api` | Representative REST flows execute | Covered by passing API suite rerun | PASS |
| AT-005 | API Integration | Supertest | Payment/profile/authorization regression | `npm run test:api` | Representative protected APIs execute | Covered by passing API suite rerun | PASS |
| AT-006 | API Coverage | nyc | API coverage report | `npm run test:api:coverage` | API lcov/text-summary generated | Statements 91.36%, branches 76.92%, functions 92.28%, lines 91.71% | PASS |
| AT-007 | Frontend Unit | Vitest + Angular TestBed | Angular component/service regression | `npm run test:frontend` | Frontend unit suite executes | 121 files, 1309 pass, 0 fail | PASS |
| AT-008 | Frontend Coverage | Vitest V8 | Frontend coverage report | `npm run test:frontend:coverage` | HTML/lcov generated | Statements 93.37%, branches 90.5%, functions 84.01%, lines 96.08% | PASS |
| AT-009 | E2E | Cypress | Registration flow | `npm run cypress:run` | Browser opens and flow executes | Full run blocked by browser connection failure | BLOCKED |
| AT-010 | E2E | Cypress | Login flow | `npm run cypress:run` | Browser login flow executes | `login.spec.ts` reached after browser abort; aggregate unavailable | BLOCKED |
| AT-011 | E2E | Cypress | Search/product flow | `npm run cypress:run` | Browser search/product flow executes | Full run aggregate unavailable | BLOCKED |
| AT-012 | E2E | Cypress | Basket and checkout flow | `npm run cypress:run` | Add item and checkout in browser | `basket.spec.ts` observed 9 passing before later browser block | PASS (observed spec) |
| AT-013 | E2E | Cypress | Authorization/security challenge flows | `npm run cypress:run` | Protected/challenge flows execute | Initial administration/B2B/complaint/contact specs had passing tests; later run blocked | PARTIAL / BLOCKED |
| AT-014 | Repeatability | Node test runner | Repeat server automation | `npm run test:server` twice | Same suite counts and status | Both runs: 419 tests, 414 pass, 0 fail, 5 skipped | PASS |
| AT-015 | CI Automation | GitHub Actions | Matrix server/API/frontend/E2E automation | CI workflow inspection | Workflow defines automated jobs | Jobs/configuration present; workflow not run locally | NOT EXECUTED |

## Notes

- These are mappings to existing automated suites, not newly fabricated test cases.
- `530` API passes and `414` server passes are test execution totals, not coverage percentages.
- Cypress aggregate result is not reported because the command was blocked by browser connection failure.