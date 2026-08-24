# System Architecture

## Portfolio map

Seven projects, six tools, four applications under test. This diagram shows how they all relate.

```mermaid
graph TB
    subgraph Apps["Applications Under Test"]
        MC["movie-catalog-api Spring Boot REST API"]
        MU["movie-catalog-ui Angular 22 front end"]
        RM["Rick & Morty API rickandmortyapi.com"]
        OH["OrangeHRM opensource-demo.orangehrmlive.com"]
    end

    subgraph Frameworks["Test Frameworks"]
        PW["playwright-ts TypeScript · Playwright"]
        AT["api-testing-ts TypeScript · Jest · Axios · AJV"]
        AJ["api-testing-java Java · REST Assured · TestNG"]
        MUT["movie-catalog-ui (self) TypeScript · Angular 22 · Vitest"]
        GA["gatling-performance-tests Java · Gatling"]
        RA["RestAssuredContractTest Java · REST Assured · TestNG"]
        SJ["selenium-java Java · Selenium · TestNG · PageFactory"]
    end

    PW -->|"E2E · a11y · visual · perf · network · auth"| MU
    AT -->|"smoke · contract · regression"| MC
    AJ -->|"smoke · contract · integration · regression"| MC
    MUT -->|"component/unit"| MU
    MU -.->|"HTTP (runtime, not test-time)"| MC
    GA -->|"load · stress · spike · soak"| MC
    RA -->|"contract · negative"| RM
    SJ -->|"E2E login · PIM · Leave"| OH
```

---

## Testing pyramid - portfolio view

Each layer of the pyramid is covered by at least one project. No layer is covered by only one tool.

```mermaid
flowchart TD
    V["🔺 Visual Regression playwright-ts - screenshot baselines OS-specific · manual trigger only"]
    P["⚡ Performance playwright-ts - timing + heap budgets per page gatling-performance-tests - load · stress · spike · soak"]
    A["♿ Accessibility playwright-ts - axe-core scans + keyboard navigation"]
    E["🌐 E2E / UI playwright-ts - movie-catalog-ui full journey: auth, list, add/edit, detail, error popup selenium-java - OrangeHRM login · PIM · Leave"]
    C["📋 Contract api-testing-ts - AJV schema files for movie-catalog-api api-testing-java - inline REST Assured assertions for movie-catalog-api RestAssuredContractTest - JSON Schema for Rick & Morty API pact-contract-tests - CDC interactions for movie-catalog-api"]
    F["🔗 API Functional api-testing-ts - CRUD + filter + negative paths api-testing-java - CRUD + studios + movies"]
    U["🧪 Unit playwright-ts - DataFactory + utility tests movie-catalog-ui - components · forms · pipes (Vitest)"]

    V --> P --> A --> E --> C --> F --> U
```

---

## playwright-ts internal architecture

The most complex project in the portfolio. All layers compose through Playwright's `test.extend()` fixture system.

```mermaid
flowchart TD
    T["Test files *.test.ts"]
    FX["Fixtures test.extend() - DI container"]
    PO["Page Objects LoginPage · ListPage · AddMoviePage MovieDetailPage · ErrorPopup · BasePage"]
    UT["Utilities ApiClient · DataFactory AccessibilityHelper · VisualHelper DebugHelper · ENV · globalSetup"]
    PW["Playwright internals Browser · BrowserContext · Page APIRequestContext"]

    T -->|"declares needed fixtures"| FX
    FX -->|"composes"| PO
    FX -->|"injects"| UT
    PO -->|"wraps"| PW
    UT -->|"uses"| PW
```

### Fixture composition

Fixtures are declared as dependencies of each other, not of the test. Most page-object fixtures (`listPage`, `addMoviePage`, `movieDetailPage`) depend only on the base `page` fixture directly, since this suite's flows don't chain through one another's set up the way a multistep checkout would. The one real dependency chain is the authenticated branch: a test that needs `loggedInAddMoviePage` automatically gets `loggedInContext` and `browser` without declaring them.

```mermaid
flowchart LR
    browser --> context --> page
    page --> loginPage
    page --> listPage
    page --> addMoviePage
    page --> movieDetailPage
    browser --> loggedInContext
    loggedInContext --> loggedInPage
    loggedInContext --> loggedInAddMoviePage
```

`loggedInContext`/`loggedInPage` are the exception - they bypass the login flow by loading a persisted storage state (`.auth/moviecatalog.json`), giving tests a pre-authenticated `ListPage` with no login overhead. `loginPage` itself still drives the real form when a test needs to exercise login directly (e.g. the one `@smoke` case that actually logs in through the UI).

---

## api-testing-ts internal architecture

```mermaid
flowchart TD
    T["Jest Test Suites smoke · contract · integration · regression"]
    AC["ApiClient Axios wrapper - typed request/response"]
    RA["Resource APIs MoviesApi · StudiosApi"]
    AJV["AJV Validator compiled JSON Schema validators"]
    CM["Custom Matcher toRespondWithin(ms)"]
    APP["movie-catalog-api Spring Boot · PostgreSQL 16"]

    T --> RA --> AC -->|HTTP| APP
    T -->|"schema assertions"| AJV
    T -->|"latency assertions"| CM
```

---

## movie-catalog-ui internal architecture

```mermaid
flowchart TD
    T["Vitest Specs one per component/service/pipe"]
    TB["Angular TestBed renders components with DI"]
    HTC["HttpTestingController intercepts every MovieService call"]
    MS["MovieService HttpClient wrapper"]
    RF["Reactive Forms FormBuilder + CustomValidator"]
    APP["movie-catalog-api real backend (not hit in tests)"]

    T -->|"renders via"| TB
    TB -->|"drives"| RF
    T -->|"asserts requests via"| HTC
    HTC -->|"intercepts"| MS
    MS -.->|"HTTP (real, dev/E2E only)"| APP
```

No test in this suite reaches the real backend - `HttpTestingController` intercepts and flushes every `MovieService` call, keeping the suite deterministic and fast. See [movie-catalog-ui strategy](../strategy/movie-catalog-ui.md) for what's covered per component.

---

## Gatling internal architecture

```mermaid
flowchart TD
    SIM["Simulation LoadSimulation · StressSimulation SpikeSimulation · SoakSimulation · BasicSimulation"]
    SC["Scenario MovieScenarios (shared login + flow, across all simulations)"]
    CFG["Config.java thresholds · base URLs · user counts"]
    GE["Gatling Engine open model injection · assertions"]
    API["movie-catalog-api Spring Boot REST API (JWT auth)"]

    SIM -->|"injects users into"| SC
    SIM -->|"reads thresholds from"| CFG
    SC -->|"HTTP requests via"| GE
    GE -->|"load against"| API
```

The scenario is the only moving part shared across simulations. Changing the load profile (ramp shape, user count, duration) is the simulation's only responsibility - test logic lives in the scenario.

---

## Design decisions

### Why fixtures over setup/teardown hooks

TestNG and JUnit use `@BeforeMethod` / `@AfterMethod` hooks that run sequentially and share state through instance variables. Playwright fixtures are lazily instantiated, scoped per-test, and compose declaratively. A test that needs `cartPage` declares it; a test that needs only `loginPage` never pays the cost of initialising `cartPage`. This eliminates the class of bug where a `@BeforeMethod` runs too much setup for tests that don't need it.

### Why separate Jest configs instead of tags

Jest's `--testPathPattern` can filter by file path, but it requires the caller to know the pattern. Four named config files (`jest.smoke.config.js`, `jest.contract.config.js`, etc.) make the intent explicit - `npm run test:smoke` is unambiguous, portable across CI systems, and doesn't require knowledge of the directory structure.

### Why Java for Gatling and REST Assured

Gatling simulations and REST Assured tests are code, not configuration files. The Java Gatling DSL and REST Assured's fluent `given/when/then` are idiomatic in a Java ecosystem. Keeping performance and contract tests in Java alongside potential backend test infrastructure avoids context-switching for teams that also maintain Java services.

### Why AJV schema files over inline assertions

Inline assertions like `expect(response.data.mid).toBe('number')` scale linearly - every new field needs a new assertion. AJV compiles a JSON Schema once and validates the entire object tree in one call. The schema file is also shareable with the backend team as a machine-readable contract document, independently of the test code.
