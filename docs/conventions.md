# Development Conventions — Pokémon Games

## 1. General Principles

- Follow `docs/architecture.md` as the source of truth.
- Implement only requirements included in the MVP backlog.
- Prefer simplicity, readability, and maintainability.
- Follow SOLID, DRY, KISS, and YAGNI principles.
- Avoid speculative abstractions.
- Do not introduce new dependencies without justification.
- Do not modify unrelated code.
- Do not implement future features prematurely.
- Keep changes small, focused, and testable.
- Never silently change documented business rules.

## 2. Language Conventions

| Context                          | Language          |
| -------------------------------- | ----------------- |
| Source code                      | English           |
| Classes, methods, variables      | English           |
| API routes                       | English           |
| Database entities and properties | English           |
| Git commits                      | English           |
| Documentation                    | English           |
| User-facing text                 | Spanish initially |

All identifiers must use descriptive names.

Avoid unexplained abbreviations.

## 3. Backend Conventions

### Technology

- C# / .NET 10
- ASP.NET Core Web API
- Entity Framework Core
- PostgreSQL
- Nullable reference types enabled
- Implicit usings enabled
- Async APIs for I/O operations

### Naming

| Element        | Convention   | Example            |
| -------------- | ------------ | ------------------ |
| Class          | PascalCase   | GameSessionService |
| Interface      | IPascalCase  | IGameRepository    |
| Method         | PascalCase   | SubmitGuessAsync   |
| Property       | PascalCase   | RemainingAttempts  |
| Local variable | camelCase    | currentRound       |
| Private field  | \_camelCase  | \_gameRepository   |
| Constant       | PascalCase   | MaximumAttempts    |
| Async method   | Async suffix | GetGameAsync       |

### Architecture

- Domain contains entities, enums, value objects, and pure business rules.
- Application contains use cases, interfaces, and application orchestration.
- Infrastructure contains EF Core, persistence, and external integrations.
- Api contains endpoints, HTTP configuration, and request/response mapping.

Do not reference infrastructure frameworks from Domain.

Do not put business rules in controllers or endpoints.

Do not create generic repositories unless they provide a clear benefit.

### Dependency Injection

- Register dependencies explicitly.
- Use constructor injection.
- Avoid service locator patterns.
- Respect DI lifetimes.
- Do not inject DbContext into singleton services.

### Asynchronous Programming

- Use async/await for database and HTTP operations.
- Propagate CancellationToken.
- Avoid `.Result`, `.Wait()`, and unnecessary Task.Run.
- Do not create async methods for purely synchronous calculations.

### Validation

Validate inputs at API boundaries and enforce invariants in the domain.

Examples:

- DisplayName must not be empty.
- A game requires 1–4 distinct players.
- A guess must contain exactly one A–Z letter.
- A completed round cannot accept guesses.
- A game cannot exceed its configured round count.

### Error Handling

- Use ProblemDetails.
- Return appropriate HTTP status codes.
- Do not return stack traces to clients.
- Do not silently swallow exceptions.
- Log unexpected failures with structured logging.

### Logging

Use ILogger<T>.

Log relevant events:

- Game created
- Round completed
- Game completed
- Game abandoned
- Catalog import completed
- Unexpected errors

Never log installation credentials or secrets.

## 4. API Conventions

Use REST-style endpoints with plural resource names.

Examples:

```text
/api/players
/api/games
/api/games/{id}
```

Use appropriate HTTP methods:

| Method | Purpose                                  |
| ------ | ---------------------------------------- |
| GET    | Read resources                           |
| POST   | Create resources or execute game actions |
| PATCH  | Partial updates                          |
| DELETE | Delete resources, if required            |

### Response Rules

- Use DTOs, not EF Core entities.
- Never expose the secret answer during an active round.
- Never expose installation credentials.
- Return UTC timestamps using ISO 8601.
- Use consistent JSON naming through ASP.NET Core defaults.
- Document endpoints using OpenAPI.

### HTTP Status Codes

| Status | Meaning                                    |
| ------ | ------------------------------------------ |
| 200    | Successful request                         |
| 201    | Resource created                           |
| 400    | Invalid input                              |
| 401    | Missing or invalid installation credential |
| 403    | Operation not permitted                    |
| 404    | Resource not found                         |
| 409    | Invalid game state transition              |
| 500    | Unexpected server error                    |

Repeated guesses should produce a consistent validation or conflict response without consuming attempts.

## 5. Database Conventions

- Use EF Core migrations.
- Use PostgreSQL as the primary database.
- Use Guid identifiers for business entities.
- Preserve integer IDs from PokéAPI for Pokémon.
- Store timestamps in UTC.
- Define foreign keys explicitly.
- Add indexes for frequently queried relationships.
- Configure unique constraints where required.
- Use transactions for multi-entity game state changes.
- Apply concurrency protection to mutable game state.

### Naming

| Element   | Convention             |
| --------- | ---------------------- |
| Entity    | Singular PascalCase    |
| DbSet     | Plural PascalCase      |
| Property  | PascalCase             |
| Migration | Descriptive PascalCase |

### Migrations

- Do not modify existing applied migrations.
- Create new migrations for schema changes.
- Do not use EnsureCreated for normal application startup.
- Avoid automatic production migrations without a controlled deployment process.

## 6. Frontend Conventions

### Technology

- React
- TypeScript strict mode
- Vite
- Tailwind CSS
- Zustand
- TanStack Query
- React Router

### Naming

| Element          | Convention       | Example          |
| ---------------- | ---------------- | ---------------- |
| React component  | PascalCase       | HangmanBoard     |
| Component file   | PascalCase.tsx   | HangmanBoard.tsx |
| Hook             | useCamelCase     | useGameSession   |
| Utility          | camelCase        | normalizeWord    |
| Type / Interface | PascalCase       | GameSessionDto   |
| Constant         | UPPER_SNAKE_CASE | MAX_PLAYERS      |

### Components

- Prefer functional components.
- Keep components focused on one responsibility.
- Extract reusable UI only when justified.
- Avoid large components with mixed business and presentation logic.
- Prefer composition over excessive prop drilling.
- Avoid `any`.
- Use semantic HTML and accessible controls.

### State Management

Use TanStack Query for server state:

- Players
- Games
- Rounds
- Scores
- Results

Use Zustand only for cross-component client UI state.

Use local component state for isolated interactions.

Do not calculate authoritative game results in the frontend.

### API Communication

- Centralize HTTP requests.
- Use typed request and response DTOs.
- Handle loading, error, and empty states.
- Invalidate or refresh queries after mutations.
- Avoid direct fetch calls inside presentation components.

### Styling

- Use Tailwind CSS.
- Use consistent spacing, typography, and colors.
- Design mobile-first.
- Avoid inline styles unless necessary.
- Support touch and keyboard interactions.
- Ensure visible focus indicators.
- Respect reduced-motion preferences.

## 7. Game Engine Conventions

The Hangman engine must be deterministic for a given input state and guess.

Separate:

1. Word normalization
2. Guess evaluation
3. Round state transitions
4. Scoring
5. Turn selection
6. Pokémon selection

The engine must not depend directly on React, HTTP, EF Core, or PokéAPI.

Random Pokémon selection must be abstracted so it can be controlled in tests.

Use named constants for game rules.

Do not duplicate scoring or turn logic in frontend components.

## 8. Testing Conventions

### Backend

Use xUnit.

Test names should describe behavior:

```text
SubmitGuess_WhenLetterIsCorrect_RevealsAllOccurrences
SubmitGuess_WhenLetterWasAlreadyUsed_DoesNotConsumeAttempt
CompleteRound_WhenSixErrorsAreReached_ReturnsLost
NextTurn_WhenRoundIsActive_ReturnsConflict
```

Prioritize domain tests before infrastructure tests.

Use integration tests for database behavior and HTTP contracts.

### Frontend

Use Vitest and React Testing Library.

Test user-observable behavior rather than implementation details.

Use Playwright for critical end-to-end flows.

### Minimum Expectations

Every new business rule must have tests.

Every bug fix should include a regression test when practical.

Do not delete or weaken tests merely to make a build pass.

## 9. Git Conventions

### Branches

```text
feature/project-foundation
feature/pokemon-catalog
feature/player-management
feature/hangman-engine
feature/local-multiplayer
feature/game-recovery
fix/duplicate-guesses
```

### Commits

Use Conventional Commits:

```text
feat: implement hangman scoring
fix: prevent duplicate guesses
test: cover multiplayer turn rotation
docs: update API architecture
refactor: extract word normalization
```

Keep commits focused.

Do not commit generated build output, secrets, or local environment files.

## 10. Configuration

- Use environment variables for environment-specific settings.
- Keep secrets out of source control.
- Provide `.env.example` files where appropriate.
- Use ASP.NET Core configuration conventions.
- Use separate development and production configurations.
- Do not hardcode database credentials or API URLs.

## 11. Agent Implementation Rules

When implementing a backlog task:

1. Read `architecture.md`.
2. Read `conventions.md`.
3. Identify the relevant backlog item.
4. Inspect existing code before modifying it.
5. Implement only the requested scope.
6. Add or update relevant tests.
7. Run available validation commands.
8. Report completed work and validation results.
9. Identify blockers or deviations explicitly.
10. Do not mark work complete if acceptance criteria are unmet.

### Agent Response Format

After each implementation task, report:

```text
Implemented:
- ...

Files changed:
- ...

Tests:
- ...

Pending:
- ...
```

Do not claim that tests passed unless they were actually executed.

## 12. Definition of Done

A task is complete when:

- Acceptance criteria are satisfied.
- Code follows these conventions.
- Relevant tests pass.
- No unrelated functionality is broken.
- No secrets are exposed.
- Documentation is updated when necessary.
- The implementation is ready for review.
