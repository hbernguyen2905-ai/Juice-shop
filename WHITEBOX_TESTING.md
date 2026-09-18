# White-box Testing

## 1. Objective

Phân tích và kiểm thử trực tiếp control flow của một slice backend có logic security, authentication/authorization, validation và environment branching. Coverage được lấy từ `nyc` chạy trên server unit suite hiện có.

## 2. Scope

Scope được giới hạn ở:

- [lib/insecurity.ts](lib/insecurity.ts): JWT helpers, sanitization, coupon validation, authenticated-user map và authorization middleware.
- [lib/utils.ts](lib/utils.ts): `getChallengeEnablementStatus` và `jwtFrom`.
- Test files liên quan: [test/server/insecurity.unit.test.ts](test/server/insecurity.unit.test.ts) và [test/server/utils.unit.test.ts](test/server/utils.unit.test.ts).

Không thay đổi testing framework, source business logic hoặc các intentional vulnerabilities của Juice Shop.

## 3. Selected Modules

| Module/File | Function | Purpose | Input | Output | Dependencies | Important conditions | Existing tests | Missing/added coverage |
|---|---|---|---|---|---|---|---|---|
| `lib/insecurity.ts` | `authorize`, `verify`, `decode` | Tạo/xác minh/đọc JWT | User payload hoặc token | JWT, boolean hoặc payload | `jsonwebtoken`, `jws`, public/private key | Token có/không hợp lệ, signature, expiry | Có trong auth/API tests | Existing tests đã cover phần lớn; middleware restore bổ sung |
| `lib/insecurity.ts` | `discountFromCoupon` | Decode và kiểm tra coupon | Optional encoded coupon | Discount hoặc `undefined` | `z85`, current date, format check | Missing, malformed, format, expired, valid | Có đầy đủ trong insecurity unit tests | Không thêm test trùng |
| `lib/insecurity.ts` | `sanitizeSecure` | Recursive HTML sanitization | HTML string | Sanitized string | `sanitize-html` | Sanitized khác input thì lặp; giống thì return | Có plain/HTML/XSS cases | Existing coverage đủ cho selected paths |
| `lib/insecurity.ts` | `isCustomer` | Kiểm tra role customer từ token | Express request | Boolean | `jwtFrom`, `verify`, `decode` | Token thiếu, role đúng/sai | Có | Existing coverage đủ |
| `lib/insecurity.ts` | `isDeluxe` | Kiểm tra role và deluxe HMAC | Express request | Boolean/falsey | JWT helpers, `deluxeToken` | Role, token tồn tại, HMAC đúng/sai | Có role/token cases | Thêm missing-token false branch |
| `lib/insecurity.ts` | `isAccounting` | Express middleware cho accounting role | Request/response/next | `next()` hoặc HTTP 403 | JWT helpers | Role accounting đúng/sai | Có | Existing coverage đủ |
| `lib/insecurity.ts` | `appendUserId` | Gắn UserId từ authenticated token | Request/response/next | Mutated body hoặc 401 | `authenticatedUsers`, `jwtFrom` | Token map hit/miss | Có | Existing coverage đủ |
| `lib/insecurity.ts` | `updateAuthenticatedUsers` | Khôi phục in-memory user từ cookie/header token | Request/response/next | Side effect, `next()` | `jsonwebtoken.verify`, token map | Cookie/header token, known/unknown, valid/invalid JWT, decoded data | Có only-next case trước phase | Thêm valid-cookie restore và invalid-cookie ignore |
| `lib/utils.ts` | `jwtFrom` | Parse Bearer header | Request headers | Token hoặc `undefined` | Không có external dependency | Header missing, malformed, Basic, Bearer | Có | Existing coverage đủ |
| `lib/utils.ts` | `getChallengeEnablementStatus` | Quyết định challenge enabled/disabled | Challenge, safety mode, environment predicates | `{ enabled, disabledBecause }` | Environment callbacks, config default | disabledEnv, Docker/Heroku/Windows, safety mode | Có matching environment cases | Thêm environment-mismatch auto branch |

## 4. Source Code Analysis

### 4.1 `updateAuthenticatedUsers`

Control flow thực tế:

```text
Request
  |
  +-- token = cookies.token || Bearer header
        |
        +-- token absent ----------------------> next()
        |
        +-- token known in authenticatedUsers -> next()
        |
        +-- token unknown
              |
              +-- jwt.verify error -----------> next(), no cookie restore
              |
              +-- verify succeeds
                    |
                    +-- decoded.data absent -> next(), no restore
                    |
                    +-- decoded.data present -> put token, set cookie, next()
```

Tests cover the no-token path, valid unknown cookie token and invalid cookie token. The decoded-without-`data` path remains outside the added slice.

### 4.2 `isDeluxe`

```text
Request
  |
  +-- Bearer token missing/invalid -> verify/decode falsey -> false
  |
  +-- token valid
        |
        +-- role != deluxe -> false
        |
        +-- role deluxe
              |
              +-- deluxeToken missing -> falsey
              |
              +-- HMAC mismatch -> false
              |
              +-- HMAC match -> true
```

### 4.3 `getChallengeEnablementStatus`

```text
Challenge
  |
  +-- no disabledEnv -> enabled
  |
  +-- safetyMode == disabled -> enabled
  |
  +-- matching Docker/Heroku/Windows environment -> disabled for environment
  |
  +-- disabledEnv + safetyMode == enabled -> disabled for Safety Mode
  |
  +-- otherwise -> enabled
```

The added test covers the final `otherwise` path when `disabledEnv= Docker`, safety mode is `auto`, and all environment predicates are false.

## 5. Control Flow Analysis

### Branch mapping

| Branch ID | Function | Condition | True path | False path | Test case |
|---|---|---|---|---|---|
| WB-B01 | `isDeluxe` | Token is valid deluxe user with matching HMAC | Return `true` | Continue role/token checks | WB-001, WB-002 |
| WB-B02 | `isDeluxe` | Request has no token | Not applicable | Return falsey | WB-003 |
| WB-B03 | `isAccounting` | Decoded role is accounting | Call `next()` | Return 403 JSON | Existing `insecurity.unit.test.ts` cases |
| WB-B04 | `appendUserId` | Token map contains user | Set `req.body.UserId`, call `next()` | Catch and return 401 | Existing `insecurity.unit.test.ts` cases |
| WB-B05 | `updateAuthenticatedUsers` | Cookie token is unknown but valid | Put decoded user and set cookie | No restore for invalid token | WB-004, WB-005 |
| WB-B06 | `getChallengeEnablementStatus` | Environment matches disabledEnv | Return environment disabled | Continue to later decisions | Existing environment cases |
| WB-B07 | `getChallengeEnablementStatus` | `disabledEnv` exists, safety mode is enabled | Return Safety Mode disabled | Final enabled path | Existing safety mode cases, WB-006 |

## 6. Statement/Line Coverage

The configured tool reports both statement coverage and line coverage. They are recorded separately.

`nyc` configuration: `package.json` `nyc` block, including `lib/*.ts`, `models/*.ts`, `routes/*.ts`, `server.ts`, reporters `lcov` and `text-summary`.

## 7. Branch Coverage

`nyc` reports branch coverage for the complete server instrumentation scope and for selected files through `lcov.info`. It does not report a separate per-test branch table, so branch IDs above are source-analysis mappings, not invented percentages.

## 8. Condition Coverage

`nyc` output used here does not provide a separate condition-coverage metric. Compound conditions were reviewed qualitatively where present, but no condition percentage is reported.

## 9. Path Coverage

Path analysis was limited to small functions:

| Path ID | Function | Conditions | Test | Executed |
|---|---|---|---|---|
| P1 | `isDeluxe` | valid deluxe role + matching HMAC | WB-001 | Yes |
| P2 | `isDeluxe` | deluxe role + invalid HMAC | WB-002 | Yes |
| P3 | `isDeluxe` | non-deluxe role | Existing test | Yes |
| P4 | `isDeluxe` | no token | WB-003 | Yes |
| P5 | `updateAuthenticatedUsers` | no token | Existing test | Yes |
| P6 | `updateAuthenticatedUsers` | unknown valid cookie token | WB-004 | Yes |
| P7 | `updateAuthenticatedUsers` | invalid cookie token | WB-005 | Yes |
| P8 | `getChallengeEnablementStatus` | mismatched environment + auto mode | WB-006 | Yes |

Full path coverage of `lib/insecurity.ts` and `lib/utils.ts` is not claimed because both modules contain many unrelated functions and external/error paths.

## 10. Existing Tests

Before additions, the selected behavior already had tests for coupon validation, sanitization, customer/deluxe/accounting roles, token map access, `appendUserId`, `jwtFrom` and matching challenge environments. The baseline server suite had `410 pass`, `0 fail`, `5 skipped`.

## 11. Additional Tests

Four tests were added:

- `WB-003`: `isDeluxe` returns false without a token.
- `WB-004`: valid unknown cookie token is restored into authenticated-user state.
- `WB-005`: invalid cookie token is ignored.
- `WB-006`: environment-mismatched challenge remains enabled in `auto` mode.

No source code was changed. The test-only additions exercise previously unhit branches identified from baseline lcov data.

## 12. Coverage Before

Command:

```text
npm run test:server:coverage
```

Tool: `nyc`.

Full configured server scope:

- Statements: `40.72% (1326/3256)`
- Branches: `45.07% (741/1644)`
- Functions: `39.13% (261/667)`
- Lines: `40.97% (1257/3068)`

Selected files:

| File | Lines | Branches | Functions |
|---|---:|---:|---:|
| `lib/insecurity.ts` | `90/94` | `47/57` | `32/33` |
| `lib/utils.ts` | `124/133` | `77/88` | `25/28` |

## 13. Coverage After

Command:

```text
npm run test:server:coverage
```

Tool: `nyc`.

Full configured server scope:

- Statements: `40.90% (1332/3256)`
- Branches: `45.68% (751/1644)`
- Functions: `39.28% (262/667)`
- Lines: `41.16% (1263/3068)`

Selected files:

| File | Lines | Branches | Functions |
|---|---:|---:|---:|
| `lib/insecurity.ts` | `94/94` | `53/57` | `33/33` |
| `lib/utils.ts` | `126/133` | `81/88` | `25/28` |

### Before/After summary

| Metric | Before | After | Difference |
|---|---:|---:|---:|
| Statements | 40.72% | 40.90% | +0.18 percentage points |
| Lines | 40.97% | 41.16% | +0.19 percentage points |
| Branches | 45.07% | 45.68% | +0.61 percentage points |
| Functions | 39.13% | 39.28% | +0.15 percentage points |

The percentages and numerator/denominator values above are copied from the actual nyc text summary. `nyc` provides statement coverage; it is not inferred from test counts.

## 14. Uncovered Code

After coverage, selected `lib/insecurity.ts` has no uncovered executable lines according to lcov, although 4 of 57 instrumented branches remain uncovered. Remaining selected-file line gaps are in `lib/utils.ts`:

- `utils.ts:71`: filesystem fallback in `getCtfKey` when `CTF_KEY` is absent.
- `utils.ts:109-111,113`: `downloadToFile` success/error behavior not exercised by this focused server suite.
- `utils.ts:170`: safety-mode branch behavior outside the selected added path.
- `utils.ts:247`: asynchronous error forwarding in `asyncHandler`.

These paths were not added in this phase because they require external/file setup or belong to broader utility coverage rather than the selected security slice.

## 15. Findings

- No application source behavior was changed.
- The original first version of the new safety-mode test had an incorrect expectation; actual control flow returned `Safety Mode` disabled for `safetyMode='enabled'`. The test was corrected to use `auto`, then passed.
- No new defect was concluded from this white-box slice.
- Existing intentional challenge behavior remains unchanged.

## 16. Limitations

- Coverage is for the full server unit instrumentation scope, not the entire repository or frontend.
- Condition coverage is `NOT AVAILABLE` as a separate nyc metric.
- Full path coverage is not claimed for either module.
- Coverage percentages are not equivalent to the Phase 2 `530` API test passes.
- The 5 skipped server tests remain environment/test-specific and were not altered.

## 17. Evidence cần chụp

| Evidence ID | Test Case | File/function | Screenshot cần chụp | Mục đích |
|---|---|---|---|---|
| WB-E01 | WB-001 to WB-006 | `lib/insecurity.ts`, `lib/utils.ts` | Source lines of selected functions | Chứng minh source slice được chọn |
| WB-E02 | WB-B01 to WB-B07 | Selected control flow | ASCII/control-flow analysis in report | Chứng minh decision points |
| WB-E03 | Baseline | `coverage/server-tests/lcov.info` and terminal summary | Before coverage output | Chứng minh metric trước bổ sung |
| WB-E04 | WB-003 to WB-006 | Test files | Server test runner output | Chứng minh test execution |
| WB-E05 | After coverage | `coverage/server-tests/lcov.info` and terminal summary | After coverage output | Chứng minh metric sau bổ sung |
| WB-E06 | WB-001/WB-004 | `isDeluxe`, `updateAuthenticatedUsers` | Passing true-branch test output | Chứng minh true path |
| WB-E07 | WB-003/WB-005/WB-006 | Selected false/fallback branches | Passing false-branch test output | Chứng minh false/fallback path |
| WB-E08 | Uncovered code | `lib/utils.ts` | lcov entries for remaining zero-hit lines | Chứng minh limitation/uncovered code |

## 18. Conclusion

Phase 3 đã thực hiện white-box analysis trên hai module backend có control flow security rõ ràng. Bốn test được bổ sung để thực thi các nhánh còn thiếu. Full server suite sau thay đổi đạt `414 pass`, `0 fail`, `5 skipped`. Coverage nyc tăng từ 40.72% lên 40.90% statements, từ 45.07% lên 45.68% branches, từ 39.13% lên 39.28% functions và từ 40.97% lên 41.16% lines.