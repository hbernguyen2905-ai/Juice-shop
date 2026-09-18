# White-box Test Cases

| ID | Module | Function | Technique | Condition/Branch | Input | Expected | Status |
|---|---|---|---|---|---|---|---|
| WB-001 | `lib/insecurity.ts` | `isDeluxe` | Branch Coverage / Path Coverage | Deluxe role and valid HMAC | Valid deluxe JWT with matching `deluxeToken` | Return `true` | PASS |
| WB-002 | `lib/insecurity.ts` | `isDeluxe` | Branch Coverage | Deluxe role but invalid HMAC | Valid JWT with `deluxeToken='invalid'` | Return `false` | PASS |
| WB-003 | `lib/insecurity.ts` | `isDeluxe` | Branch Coverage / Path Coverage | No token branch | Request without authorization header | Return falsey | PASS |
| WB-004 | `lib/insecurity.ts` | `updateAuthenticatedUsers` | Branch Coverage / Path Coverage | Unknown valid cookie token | Cookie contains valid JWT with `data.id=4` | Add user to map and set cookie | PASS |
| WB-005 | `lib/insecurity.ts` | `updateAuthenticatedUsers` | Branch Coverage / Error Path | Invalid JWT verification | Cookie contains `invalid-token` | Call next, do not restore user/cookie | PASS |
| WB-006 | `lib/utils.ts` | `getChallengeEnablementStatus` | Branch Coverage / Path Coverage | `disabledEnv` mismatch, auto mode | Docker disabledEnv, all environment callbacks false, mode `auto` | Challenge remains enabled | PASS |
| WB-007 | `lib/insecurity.ts` | `isAccounting` | Branch Coverage | Accounting role vs other role | Accounting/admin JWT | `next()` for accounting; 403 otherwise | PASS, existing |
| WB-008 | `lib/insecurity.ts` | `appendUserId` | Branch Coverage / Error Path | Known token vs missing token | Token map hit/miss | Add UserId or return 401 | PASS, existing |
| WB-009 | `lib/insecurity.ts` | `discountFromCoupon` | Statement/Branch Coverage | Missing, malformed, expired, valid coupon | Representative coupon values | `undefined` or discount | PASS, existing |
| WB-010 | `lib/insecurity.ts` | `sanitizeSecure` | Statement/Branch Coverage | Input unchanged vs recursive sanitization | Plain/harmless/malicious HTML | Safe sanitized output | PASS, existing |
| WB-011 | `lib/utils.ts` | `jwtFrom` | Branch Coverage | Bearer, missing, malformed, Basic header | Authorization header variants | Token or `undefined` | PASS, existing |
| WB-012 | `lib/utils.ts` | `getChallengeEnablementStatus` | Branch Coverage | Environment match and safety modes | Docker/Heroku/Windows and enabled/disabled/auto | Correct enabled/disabled result | PASS, existing |

## Execution

Command:

```text
npm run test:server
```

Result after additional tests:

- Total test executions: `419` (`414 pass` + `5 skipped`)
- PASS: `414`
- FAIL: `0`
- SKIPPED: `5`

Coverage command:

```text
npm run test:server:coverage
```

The coverage tool reports statements, branches, functions and lines. Separate condition coverage is `NOT AVAILABLE`.

The four newly added cases are WB-003, WB-004, WB-005 and WB-006. The remaining cases document existing white-box coverage and were not duplicated.