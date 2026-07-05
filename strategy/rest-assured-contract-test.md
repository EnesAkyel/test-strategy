# API Contract Testing Strategy - REST Assured

**Project:** [`RestAssuredContractTest`](https://github.com/EnesAkyel/RestAssuredContractTest)  
**Stack:** Java · REST Assured · TestNG · Allure · JSON Schema Validator  
**Target:** Rick & Morty API (`https://rickandmortyapi.com/api`)

---

## Purpose

Contract tests verify that a third-party or shared API honors the shape and behavior it has documented. Unlike functional tests, contract tests are not concerned with the business logic behind the response - they are concerned with the **structure, types, and status codes** that consumers depend on.

This project treats the Rick & Morty API as a dependency: if its response shape changes in a way that would break a consumer, these tests catch it. The same pattern applies directly to any internal API that multiple teams consume.

---

## What Is Covered

### Endpoints under contract

| Endpoint                       | Test class                | What is validated                                               |
|--------------------------------|---------------------------|-----------------------------------------------------------------|
| `GET /character`               | `GetAllCharactersTest`    | Schema, pagination metadata, first result fields, response time |
| `GET /character?name=&status=` | `FilterCharactersTest`    | Schema, filter correctness (every result matches both params)   |
| `GET /location/:id`            | `GetASingleLocationTest`  | Schema, field presence                                          |
| `GET /episode` (batch)         | `GetMultipleEpisodesTest` | Multi-ID batch response, schema                                 |
| Non-existent IDs               | `NegativeContractTest`    | 404 status, exact error message body                            |

### Negative contract testing

`NegativeContractTest` covers error responses as part of the contract - not just happy paths. The assertions verify that:
- A non-existent character ID returns `404` with `{ "error": "Character not found" }`
- A non-existent location ID returns `404` with `{ "error": "Location not found" }`
- A non-existent episode ID returns `404` with `{ "error": "Episode not found" }`
- A filter that matches no characters returns `404` with `{ "error": "There is nothing here" }`

Error response shapes are part of the contract. If the API changes `"Character not found"` to `"character_not_found"`, a consumer that parses that message will break. These tests catch it.

---

## Schema Validation Approach

JSON schema files are stored in `src/test/resources/` and loaded at runtime via REST Assured's `matchesJsonSchemaInClasspath()`. This means the schema is the single source of truth for what a valid response looks like.

Advantages over field-by-field assertions:

- **Completeness** - all fields in the schema are validated in one call. Adding a required field to the schema automatically covers it in every test that validates that response type.
- **Portability** - schema files can be shared with the API team as the agreed contract document.
- **Readability** - the test body stays clean; structural validation is delegated to the schema file.

---

## Parameterised Tests

`FilterCharactersTest` uses TestNG's `@DataProvider` to run the same filter validation against multiple name/status combinations:

```
("rick",  "alive")
("morty", "alive")
("beth",  "alive")
```

This ensures filter logic works correctly for different inputs without duplicating test code. Adding a new combination is a one-line change in the data provider.

---

## Response Time Assertions

Every test asserts `time(lessThan(3000L))`. This is a first-class contract assertion, not an afterthought. If the API begins responding slowly - due to infrastructure changes or increased load - tests fail with a clear timing message before it impacts consumers.

---

## Tool Choices

**REST Assured over Playwright APIRequestContext** - REST Assured's fluent `given/when/then` DSL is the industry standard for Java API testing. Its `matchesJsonSchemaInClasspath()` integration and Hamcrest matcher library make JSON contract assertions concise and expressive. For Java-based projects, it is the natural choice.

**TestNG over JUnit** - TestNG's `@DataProvider` is built-in and requires no additional annotations or libraries. Its parallel execution and test grouping capabilities are also useful for larger suites.

**Allure over TestNG's default reporter** - Allure captures `@Feature` and `@Description` annotations and produces an interactive HTML report with pass/fail history. For a contract test suite where the audience may include non-engineers, the visual report is more accessible than a text log.
