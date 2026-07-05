# System Architecture

## Portfolio map

Six projects, five tools, four applications under test. This diagram shows how they all relate.

```mermaid
graph TB
    subgraph Apps["Applications Under Test"]
        SD["SauceDemo saucedemo.com"]
        JP["JSONPlaceholder jsonplaceholder.typicode.com"]
        MC["movie-catalog-api Spring Boot REST API"]
        RM["Rick & Morty API rickandmortyapi.com"]
        OH["OrangeHRM opensource-demo.orangehrmlive.com"]
    end

    subgraph Frameworks["Test Frameworks"]
        PW["playwright-ts TypeScript · Playwright"]
        AT["api-testing-ts TypeScript · Jest · Axios · AJV"]
        AJ["api-testing-java Java · REST Assured · TestNG"]
        GA["gatling-performance-tests Java · Gatling"]
        RA["RestAssuredContractTest Java · REST Assured · TestNG"]
        SJ["selenium-java Java · Selenium · TestNG · PageFactory"]
    end

    PW -->|"E2E · a11y · visual · perf · auth"| SD
    PW -->|"API functional · hybrid API+UI"| JP
    AT -->|"smoke · contract · regression"| MC
    AJ -->|"smoke · contract · integration · regression"| MC
    GA -->|"load · stress · spike · soak"| JP
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
    E["🌐 E2E / UI playwright-ts - SauceDemo full checkout, cart, auth selenium-java - OrangeHRM login · PIM · Leave"]
    C["📋 Contract api-testing-ts - AJV schema files for movie-catalog-api api-testing-java - inline REST Assured assertions for movie-catalog-api RestAssuredContractTest - JSON Schema for Rick & Morty API playwright-ts - inline response shape assertions"]
    F["🔗 API Functional api-testing-ts - CRUD + filter + negative paths api-testing-java - CRUD + studios + movies playwright-ts - JSONPlaceholder hybrid flows"]
    U["🧪 Unit playwright-ts - DataFactory + utility tests"]

    V --> P --> A --> E --> C --> F --> U
```

---

## playwright-ts internal architecture

The most complex project in the portfolio. All layers compose through Playwright's `test.extend()` fixture system.

```mermaid
flowchart TD
    T["Test files *.test.ts"]
    FX["Fixtures test.extend() - DI container"]
    PO["Page Objects LoginPage · InventoryPage CartPage · CheckoutPage · BasePage"]
    UT["Utilities ApiClient · DataFactory AccessibilityHelper · VisualHelper DebugHelper · ENV · globalSetup"]
    PW["Playwright internals Browser · BrowserContext · Page APIRequestContext"]

    T -->|"declares needed fixtures"| FX
    FX -->|"composes"| PO
    FX -->|"injects"| UT
    PO -->|"wraps"| PW
    UT -->|"uses"| PW
```

### Fixture composition

Fixtures are declared as dependencies of each other, not of the test. A test that needs `checkoutPage` automatically gets `inventoryPage`, `cartPage`, `loginPage`, `page`, and `browser` - without declaring them.

```mermaid
flowchart LR
    browser --> context --> page
    page --> loginPage
    loginPage --> inventoryPage
    inventoryPage --> cartPage
    cartPage --> checkoutPage
    browser --> loggedInPage
```

`loggedInPage` is the exception - it bypasses the login flow by loading a persisted storage state (`.auth/sauce.json`), giving tests a pre-authenticated `InventoryPage` with no login overhead.

---

## api-testing-ts internal architecture

```mermaid
flowchart TD
    T["Jest Test Suites smoke · contract · integration · regression"]
    AC["ApiClient Axios wrapper - typed request/response"]
    RA["Resource APIs MoviesApi · StudiosApi"]
    AJV["AJV Validator compiled JSON Schema validators"]
    CM["Custom Matcher toRespondWithin(ms)"]
    APP["movie-catalog-api Spring Boot · H2 / PostgreSQL"]

    T --> RA --> AC -->|HTTP| APP
    T -->|"schema assertions"| AJV
    T -->|"latency assertions"| CM
```

---

## Gatling internal architecture

```mermaid
flowchart TD
    SIM["Simulation LoadSimulation · StressSimulation SpikeSimulation · SoakSimulation · BasicSimulation"]
    SC["Scenario PostScenarios.browsePostsFlow (shared across all simulations)"]
    CFG["Config.java thresholds · base URLs · user counts"]
    GE["Gatling Engine open model injection · assertions"]
    API["JSONPlaceholder API https://jsonplaceholder.typicode.com"]

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
