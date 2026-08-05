# Coverage Matrix

This matrix maps every project in the portfolio to the test types it covers, the application it targets, and the tool it uses. Read it as a checklist: if a cell is empty, that combination is either out of scope or handled by a different layer.

---

## By test type

| Test Type                |       playwright-ts        |    api-testing-ts    |   api-testing-java   |     movie-catalog-ui     | gatling-performance-tests | k6-performance-tests | RestAssuredContractTest | pact-contract-tests |  selenium-java  |
|--------------------------|:--------------------------:|:--------------------:|:--------------------:|:------------------------:|:-------------------------:|:--------------------:|:-----------------------:|:-------------------:|:---------------:|
| Unit                     | ✅ DataFactory, utilities  |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Component                |             -              |          -           |          -           |   ✅ Vitest + TestBed    |             -             |          -           |            -            |          -          |        -        |
| API functional           |     ✅ JSONPlaceholder     | ✅ movie-catalog-api | ✅ movie-catalog-api |            -             |             -             |          -           |            -            |          -          |        -        |
| API contract             | ✅ response shape (inline) | ✅ AJV schema files  | ✅ inline assertions |            -             |             -             |          -           |  ✅ JSON Schema files   |   ✅ Pact CDC V4    |        -        |
| E2E / UI                 |        ✅ SauceDemo        |          -           |          -           | ⏳ planned (Playwright)  |             -             |          -           |            -            |          -          |  ✅ OrangeHRM   |
| Form validation          |             -              |          -           |          -           | ✅ client + server-side  |             -             |          -           |            -            |          -          |        -        |
| Accessibility            |   ✅ axe-core, keyboard    |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Visual regression        |  ✅ screenshot baselines   |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Network mocking          |  ✅ request interception   |          -           |          -           | ✅ HttpTestingController |             -             |          -           |            -            |          -          |        -        |
| Auth persistence         |      ✅ storage state      |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Performance (functional) |  ✅ timing + heap budgets  |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Performance (load)       |             -              |          -           |          -           |            -             |        ✅ Gatling         |        ✅ k6         |            -            |          -          |        -        |
| Performance (stress)     |             -              |          -           |          -           |            -             |        ✅ Gatling         |        ✅ k6         |            -            |          -          |        -        |
| Performance (spike)      |             -              |          -           |          -           |            -             |        ✅ Gatling         |        ✅ k6         |            -            |          -          |        -        |
| Performance (soak)       |             -              |          -           |          -           |            -             |        ✅ Gatling         |          -           |            -            |          -          |        -        |
| Negative / error paths   | ✅ login, form validation  |   ✅ 4xx responses   |   ✅ 404 responses   | ✅ form + server errors  |             -             |     ✅ 404 check     |  ✅ 404 + error bodies  | ✅ 404 interactions | ✅ login errors |
| Cross-browser            |   ✅ Chromium + Firefox    |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| CI / automated           |             ✅             |          ✅          |          ✅          |            ✅            |            ✅             |          ✅          |           ✅            |         ✅          |       ✅        |

---

## By application

| Application                                                         | What it is                                            | Covered by                                                                                                                                                                                                                |
|---------------------------------------------------------------------|-------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [SauceDemo](https://www.saucedemo.com)                              | Public demo e-commerce UI                             | `playwright-ts` - E2E, a11y, visual, performance, auth                                                                                                                                                                    |
| [JSONPlaceholder](https://jsonplaceholder.typicode.com)             | Public REST API (posts, users, todos)                 | `playwright-ts` (hybrid API+UI), `gatling-performance-tests` (load)                                                                                                                                                       |
| [movie-catalog-api](https://github.com/EnesAkyel/movie-catalog-api) | Self-owned Spring Boot REST API                       | `api-testing-ts` - smoke, contract, regression; `api-testing-java` - smoke, contract, integration, regression; `k6-performance-tests` - load, stress, spike; `pact-contract-tests` - CDC consumer + provider verification |
| [Rick & Morty API](https://rickandmortyapi.com)                     | Public REST API (characters, locations, episodes)     | `RestAssuredContractTest` - contract + negative tests                                                                                                                                                                     |
| [OrangeHRM](https://opensource-demo.orangehrmlive.com)              | Open-source HR management demo                        | `selenium-java` - login, PIM, Leave module                                                                                                                                                                                |
| [movie-catalog-ui](https://github.com/EnesAkyel/movie-catalog-ui)   | Self-owned Angular 22 front end for movie-catalog-api | `movie-catalog-ui` - component/unit tests (Vitest); planned target for a future Playwright E2E suite                                                                                                                      |

---

## By tool

| Tool                            | Language   | Projects                                      | Primary strength                                                                             |
|---------------------------------|------------|-----------------------------------------------|----------------------------------------------------------------------------------------------|
| Playwright                      | TypeScript | `playwright-ts`                               | Auto-waiting, multi-browser, built-in API testing, visual/a11y                               |
| Jest + Axios + AJV              | TypeScript | `api-testing-ts`                              | Typed API client, schema-file contract validation, suite separation                          |
| Gatling                         | Java       | `gatling-performance-tests`                   | Code-first load profiles, simulation composability, HTML reports                             |
| k6                              | TypeScript | `k6-performance-tests`                        | TypeScript-native, esbuild bundled, per-scenario thresholds, group-level metrics             |
| REST Assured + TestNG           | Java       | `RestAssuredContractTest`, `api-testing-java` | Fluent API DSL, JSON schema classpath validation, data providers                             |
| Selenium + PageFactory + TestNG | Java       | `selenium-java`                               | @FindBy page objects, ThreadLocal driver, AspectJ Allure steps, CI-enabled                   |
| Pact + Jest + ts-jest           | TypeScript | `pact-contract-tests`                         | Consumer-driven contracts, Pact V4 interaction matchers, provider state handlers, two-job CI |
| Vitest + Angular TestBed        | TypeScript | `movie-catalog-ui`                            | Fast Node-based component tests, HttpTestingController, fake-timer debounce testing          |

---

## Coverage gaps (known and accepted)

| Gap                               | Reason accepted                                                                                                                      |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------|
| Security testing                  | Requires dedicated tooling (OWASP ZAP, Burp Suite) and a separate engagement - out of scope for functional suites                    |
| Mobile native apps                | All targets are web applications; responsive layout is covered via viewport config in Playwright                                     |
| Database / data integrity         | No direct DB access to the demo applications; API layer covers data contracts at the HTTP boundary                                   |
| Load testing for SauceDemo        | Third-party demo site - generating synthetic load against it would be irresponsible                                                  |
| Distributed load generation       | Both Gatling and k6 run from a single CI runner; production-scale multi-injector runs require Gatling Enterprise or Grafana Cloud k6 |
| WebKit (Safari) in CI             | Added to the nightly matrix in `playwright-ts`; not in the PR regression suite to keep CI fast                                       |
| E2E coverage for movie-catalog-ui | Component/unit layer only for now; Playwright E2E suite is planned but not yet built - see [Risk Areas](risk-areas.md)               |
