# pact-contract-tests Strategy

Consumer-driven contract (CDC) testing for `movie-catalog-api` using [Pact](https://docs.pact.io/).

---

## What CDC adds that schema validation does not

The portfolio already has three layers of contract coverage for `movie-catalog-api`:

| Layer           | Project               | What it checks                                                   |
|-----------------|-----------------------|------------------------------------------------------------------|
| Schema (inline) | `api-testing-ts`      | AJV schema files validate response shape per endpoint            |
| Schema (inline) | `api-testing-java`    | REST Assured `matchesJsonSchemaInClasspath` assertions           |
| CDC             | `pact-contract-tests` | Consumer-defined interactions verified against the real provider |

Schema validation answers "does the response match this shape?" CDC answers a different question: **"does the provider still satisfy every interaction the consumer has declared it depends on?"**

The key difference is directionality. In CDC, the consumer publishes a contract (a pact file) that says "I call this endpoint with these parameters and expect this structure." The provider then verifies it can fulfil that contract. If the provider changes a field name or removes an endpoint, verification fails - even if no schema file was updated to reflect the change.

---

## Interaction design

Eight interactions cover the main read paths of `movie-catalog-api`:

| Interaction           | Method | Path                          | Provider state                    |
|-----------------------|--------|-------------------------------|-----------------------------------|
| List all movies       | GET    | `/api/v1/movies`              | movies exist                      |
| Filter by genre       | GET    | `/api/v1/movies?genre=Action` | action movies exist               |
| Filter by rating      | GET    | `/api/v1/movies?rating=PG-13` | PG-13 movies exist                |
| Get movie by ID       | GET    | `/api/v1/movie/1001`          | movie with ID 1001 exists         |
| Non-existent movie    | GET    | `/api/v1/movie/9999`          | movie with ID 9999 does not exist |
| List all studios      | GET    | `/api/v1/studios`             | studios exist                     |
| Movies by studio      | GET    | `/api/v1/studios/1/movies`    | studio 1 has movies               |
| Studio with no movies | GET    | `/api/v1/studios/99/movies`   | studio 99 has no movies           |

Write operations (POST, PUT, DELETE) are deliberately excluded. CDC is most valuable for the read paths that consumers depend on for data; write operations are better covered by the functional integration tests in `api-testing-ts` and `api-testing-java`.

---

## Matcher strategy

All interactions use Pact V4 matchers from `MatchersV3`:

- **`like()`** - type matcher. The value in the pact file is an example; the real response just needs to match the type. Used on the full response body so example data doesn't hard-code into the contract.
- **`eachLike()`** - array type matcher. Asserts the response contains at least one element of the given shape. Used on `content` arrays in paginated responses.
- **`integer()`** / **`decimal()`** / **`string()`** - primitive type matchers. Pinpoints the type of each field without coupling the contract to a specific value.

This combination means the contract captures the **structure** the consumer needs without being brittle to data changes in the provider's seed dataset.

---

## Provider state strategy

Each interaction declares a `given(...)` state (e.g. `"movie with ID 1001 exists"`). The provider verification spec registers a handler for every state. All handlers are currently no-ops because the Flyway-seeded dataset satisfies every interaction at startup without any setup logic.

The handlers are still registered explicitly - they are the single place to add real database setup if the seed data ever changes or if the project moves away from in-memory storage.

---

## Tool choice

### Pact vs JSON schema validation

|                | Pact CDC                                       | JSON schema                      |
|----------------|------------------------------------------------|----------------------------------|
| Direction      | Consumer declares what it needs                | Provider defines what it returns |
| Failure signal | "Consumer X is broken by this provider change" | "Response does not match schema" |
| Coupling       | Consumer owns the contract                     | Schema lives in the test project |
| Setup cost     | Mock server + two-stage CI                     | Schema file + single assertion   |

Both are useful; they answer different questions. This project complements `api-testing-ts` and `api-testing-java` rather than replacing them.

### Pact vs Spring Cloud Contract

Spring Cloud Contract requires the contract to live in the provider's repository and generates both consumer stubs and provider tests from it. That is the right model when both sides of the API are owned by the same team. This portfolio uses Pact because the consumer is an external test project - the consumer-owns-the-contract model is the better fit.

---

## What is not covered

- **Write operations** - POST/PUT/DELETE interactions are not in scope. Functional correctness of writes is covered by `api-testing-ts` and `api-testing-java`.
- **Pact Broker** - no broker is used. The pact file is passed between CI jobs via GitHub Actions artifacts and committed to the repository. A broker would add value at scale (multiple consumers, can-i-deploy checks) but is unnecessary for a single consumer–provider pair.
- **Authentication** - the current API has no auth. Once the JWT login feature is implemented, a login interaction and `Authorization` header will need to be added to all interactions.
