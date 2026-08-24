# Coverage Matrix

This matrix maps every project in the portfolio to the test types it covers, the application it targets, and the tool it uses. Read it as a checklist: if a cell is empty, that combination is either out of scope or handled by a different layer.

---

## By test type

| Test Type                |         playwright-ts          |    api-testing-ts    |   api-testing-java   |     movie-catalog-ui     | gatling-performance-tests | k6-performance-tests | RestAssuredContractTest | pact-contract-tests |  selenium-java  |
|--------------------------|:------------------------------:|:--------------------:|:--------------------:|:------------------------:|:-------------------------:|:--------------------:|:-----------------------:|:-------------------:|:---------------:|
| Unit                     |   ✅ DataFactory, utilities    |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Component                |               -                |          -           |          -           |   ✅ Vitest + TestBed    |             -             |          -           |            -            |          -          |        -        |
| API functional           |               -                | ✅ movie-catalog-api | ✅ movie-catalog-api |            -             |             -             |          -           |            -            |          -          |        -        |
| API contract             |               -                | ✅ AJV schema files  | ✅ inline assertions |            -             |             -             |          -           |  ✅ JSON Schema files   |   ✅ Pact CDC V4    |        -        |
| E2E / UI                 |      ✅ movie-catalog-ui       |          -           |          -           |            -             |             -             |          -           |            -            |          -          |  ✅ OrangeHRM   |
| Form validation          |    ✅ client + server-side     |          -           |          -           | ✅ client + server-side  |             -             |          -           |            -            |          -          |        -        |
| Accessibility            |     ✅ axe-core, keyboard      |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Visual regression        |   ✅ screenshot baseline (1)   |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Network mocking          |    ✅ request interception     |          -           |          -           | ✅ HttpTestingController |             -             |          -           |            -            |          -          |        -        |
| Auth persistence         |        ✅ storage state        |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Performance (functional) |    ✅ timing + heap budgets    |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| Performance (load)       |               -                |          -           |          -           |            -             |        ✅ Gatling         |        ✅ k6         |            -            |          -          |        -        |
| Performance (stress)     |               -                |          -           |          -           |            -             |        ✅ Gatling         |        ✅ k6         |            -            |          -          |        -        |
| Performance (spike)      |               -                |          -           |          -           |            -             |        ✅ Gatling         |        ✅ k6         |            -            |          -          |        -        |
| Performance (soak)       |               -                |          -           |          -           |            -             |        ✅ Gatling         |          -           |            -            |          -          |        -        |
| Negative / error paths   |   ✅ login, form validation    |   ✅ 4xx responses   |   ✅ 404 responses   | ✅ form + server errors  |             -             |     ✅ 404 check     |  ✅ 404 + error bodies  | ✅ 404 interactions | ✅ login errors |
| Cross-browser            | ✅ Chromium + Firefox + WebKit |          -           |          -           |            -             |             -             |          -           |            -            |          -          |        -        |
| CI / automated           |               ✅               |          ✅          |          ✅          |            ✅            |            ✅             |          ✅          |           ✅            |         ✅          |       ✅        |

---

## By application

| Application                                                         | What it is                                            | Covered by                                                                                                                                                                                                                                                                         |
|---------------------------------------------------------------------|-------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| [movie-catalog-api](https://github.com/EnesAkyel/movie-catalog-api) | Self-owned Spring Boot REST API with JWT auth         | `api-testing-ts` - smoke, contract, regression; `api-testing-java` - smoke, contract, integration, regression; `k6-performance-tests` - load, stress, spike; `gatling-performance-tests` - load, stress, spike, soak; `pact-contract-tests` - CDC consumer + provider verification |
| [Rick & Morty API](https://rickandmortyapi.com)                     | Public REST API (characters, locations, episodes)     | `RestAssuredContractTest` - contract + negative tests                                                                                                                                                                                                                              |
| [OrangeHRM](https://opensource-demo.orangehrmlive.com)              | Open-source HR management demo                        | `selenium-java` - login, PIM, Leave module                                                                                                                                                                                                                                         |
| [movie-catalog-ui](https://github.com/EnesAkyel/movie-catalog-ui)   | Self-owned Angular 22 front end for movie-catalog-api | `movie-catalog-ui` - component/unit tests (Vitest); `playwright-ts` - full E2E suite (auth, list, add/edit, detail, error popup, a11y, network mocking, performance, cross-cutting NFRs)                                                                                           |

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

| Gap                                    | Reason accepted                                                                                                                                                                                         |
|----------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Security testing                       | Requires dedicated tooling (OWASP ZAP, Burp Suite) and a separate engagement - out of scope for functional suites                                                                                       |
| Mobile native apps                     | All targets are web applications; responsive layout is covered via viewport config in Playwright                                                                                                        |
| Database / data integrity              | No direct DB access to the demo applications; API layer covers data contracts at the HTTP boundary                                                                                                      |
| Distributed load generation            | Both Gatling and k6 run from a single CI runner; production-scale multi-injector runs require Gatling Enterprise or Grafana Cloud k6                                                                    |
| Full WCAG audit for movie-catalog-ui   | `playwright-ts`'s A11Y-01–04 cover critical-impact axe-core violations on the four main pages, not every WCAG success criterion or every impact level - see [Risk Areas](risk-areas.md)                 |
| Load/soak testing for movie-catalog-ui | `playwright-ts`'s PERF-01–03 are single-run regression budgets, not a load test; that belongs to `gatling-performance-tests` / `k6-performance-tests`, both of which already target `movie-catalog-api` |
