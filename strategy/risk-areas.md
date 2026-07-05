# Risk Areas

Risk assessment across the full portfolio. Each area is rated by **likelihood** (how often it breaks), **impact** (what happens when it does), and **coverage** (how well the test suite addresses it).

---

## Risk ratings

| Rating    | Likelihood                                                   | Impact                                            |
|-----------|--------------------------------------------------------------|---------------------------------------------------|
| 🔴 High   | Breaks regularly or with any change in the area              | User-facing failure, data loss, or revenue impact |
| 🟡 Medium | Breaks occasionally, usually after a specific type of change | Degraded UX or partial feature failure            |
| 🟢 Low    | Breaks rarely, usually only after significant refactoring    | Minor or cosmetic issue                           |

---

## Risk register

### Authentication and session management - 🔴 High

**Why it breaks:** Login flows are touched by security changes, dependency upgrades, and infrastructure changes. A broken login blocks every test that requires an authenticated session.

**Impact:** Complete loss of access to authenticated functionality. For an e-commerce app, this means no purchases, no account management, no order history.

**Coverage:** `playwright-ts` - login positive and negative paths, session persistence via storage state reuse, cookie validation in `authPersistence.test.ts`. Auth is the first thing to smoke-test on every deploy.

---

### Checkout and payment flow - 🔴 High

**Why it breaks:** Multi-step flows have more surface area. A change to form validation, navigation, or state management in any step silently breaks the one after it.

**Impact:** Direct revenue loss. A broken checkout is the highest-severity UI bug possible in an e-commerce application.

**Coverage:** `playwright-ts` - full E2E checkout flow with `test.step()` for granular failure localization, negative form validation for all required fields, cart state tests.

---

### API contract drift - 🔴 High

**Why it breaks:** APIs change without coordinating with consumers. Field renames, type changes, added required fields, and changed error message strings are all silent breaking changes from the consumer's perspective.

**Impact:** Consumer applications crash or behave incorrectly. The failure is often discovered in production rather than in development.

**Coverage:**
- `api-testing-ts` - AJV schema files validate every field in every response from `movie-catalog-api`
- `RestAssuredContractTest` - JSON Schema classpath files validate Rick & Morty API responses and error bodies
- `playwright-ts` - inline response shape assertions in API tests

---

### Performance degradation - 🟡 Medium

**Why it breaks:** Slow queries, unoptimized loops, missing indexes, or dependency upgrades silently increase response times. Functional tests pass because the right data comes back - just slowly.

**Impact:** Poor user experience, SEO impact, SLA breaches. Degradation is often gradual and goes undetected until users complain.

**Coverage:**
- `playwright-ts` - Navigation Timing API asserts DOMContentLoaded and load event budgets; `performance.memory` asserts JS heap budget (Chromium); end-to-end checkout duration threshold
- `gatling-performance-tests` - max response time, 95th percentile, and error rate thresholds enforced across load / stress / spike / soak profiles

---

### Visual regressions - 🟡 Medium

**Why it breaks:** CSS changes, dependency upgrades (component libraries, fonts), and browser rendering updates introduce pixel-level differences that functional tests cannot see.

**Impact:** Broken layouts, invisible text, overlapping elements. These are caught by users before automated functional tests notice anything.

**Coverage:** `playwright-ts` - full-page and element-level screenshot baselines with `toHaveScreenshot()`. OS-specific baselines (Darwin / Linux) prevent false positives from platform rendering differences. `maxDiffPixelRatio` tolerance handles sub-pixel rendering noise.

**Risk:** Visual tests are excluded from the automated CI regression suite because they require committed baseline files. Intentional UI changes need a manual baseline update via the `update-snapshots` workflow. Unmaintained baselines become a source of false negatives.

---

### Accessibility regressions - 🟡 Medium

**Why it breaks:** New components added without ARIA attributes, color contrast changes, focus order disrupted by layout changes.

**Impact:** Excludes users with disabilities. Also, a legal compliance risk under WCAG 2.1 / ADA / EN 301 549.

**Coverage:** `playwright-ts` - axe-core scans on login, inventory, cart, and checkout pages with impact-level filtering. Keyboard Tab order and keyboard-only login flow tested explicitly.

**Limitation:** axe-core catches automatable violations only - roughly 30–40% of WCAG issues. Manual a11y review is required for the remainder.

---

### Cross-browser compatibility - 🟡 Medium

**Why it breaks:** CSS properties, Web APIs, and JavaScript features behave differently across browser engines. A test that passes on Chromium can fail on Firefox or Safari due to rendering or API differences.

**Impact:** Features are broken for a subset of users depending on their browser.

**Coverage:** `playwright-ts` CI regression runs on Chromium. Nightly scheduled run adds Firefox. `performance.memory` tests are skipped on non-Chromium browsers (`test.skip(browserName !== 'chromium')`) because the API is Chromium-only. WebKit is not currently in CI.

---

### Test data pollution - 🟡 Medium

**Why it breaks:** Tests that create data without cleaning it up leave side effects that cause subsequent tests to find unexpected state. Particularly dangerous in parallel runs.

**Impact:** Intermittent failures that are hard to reproduce - the failure depends on execution order or concurrency.

**Coverage:**
- `playwright-ts` - Faker.js generates unique data per run; no shared state between tests
- `api-testing-ts` - `beforeAll` deletes test records if they exist; `afterAll` cleans up after every write test
- Fixed ID ranges (`5000+`) for write tests in `api-testing-ts` avoid colliding with seeded fixture data

---

### Third-party API availability - 🟡 Medium

**Why it breaks:** Public APIs (JSONPlaceholder, Rick & Morty) have no SLA. Downtime or rate limiting causes test failures that are not caused by the code under test.

**Impact:** False failures in CI, blocked pipelines, developer distrust of the test suite.

**Coverage:** `playwright-ts` global setup checks HTTP reachability for both target URLs before any test starts and fails with a clear message if either is unreachable. The nightly regression creates a GitHub issue on failure to distinguish infrastructure failures from test failures.

---

### Flaky tests - 🟢 Low (contained)

**Why it happens:** Timing-dependent assertions, shared state, or environment instability produce intermittent failures.

**Impact:** Developer distrust of the suite. A flaky test that sometimes fails becomes noise - developers start ignoring failures.

**Coverage:** `playwright-ts` enforces `playwright/no-wait-for-timeout` via ESLint (hardcoded waits are the primary flakiness source). CI retries failed tests twice before counting them as failures. Intermittently failing tests are quarantined with a `@flaky` tag and run in a separate project.

---

## What is not covered (and why)

| Risk area                                              | Decision                                                                                                                |
|--------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| Security vulnerabilities (XSS, SQLi, auth bypass)      | Requires dedicated security testing tools and a separate engagement. Not the responsibility of a functional test suite. |
| Data privacy / PII handling                            | Requires access to backend data stores and legal/compliance review. Out of scope.                                       |
| Infrastructure failures (DNS, CDN, network partitions) | Chaos engineering scope. Requires infrastructure access and a production-like environment.                              |
| Mobile native behaviour                                | All targets are web applications. Responsive layout is covered via Playwright viewport config.                          |
