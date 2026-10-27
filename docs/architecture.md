# Architecture — Pokémon Games

## 1. Overview

Pokémon Games is an extensible web-based minigame platform.

The MVP implements Pokémon Hangman, a turn-based word guessing game using the first 150 Pokémon from PokéAPI.

The application supports single-player and local multiplayer sessions using one shared device.

The architecture must support future games and real-time multiplayer without introducing unnecessary complexity into the MVP.

## 2. Technology Stack

| Layer             | Technology                      |
| ----------------- | ------------------------------- |
| Frontend          | React, TypeScript, Vite         |
| Styling           | Tailwind CSS                    |
| Client UI State   | Zustand                         |
| Server State      | TanStack Query                  |
| Backend           | ASP.NET Core Web API, .NET 10   |
| ORM               | Entity Framework Core           |
| Database          | PostgreSQL                      |
| External Data     | PokéAPI                         |
| Backend Testing   | xUnit                           |
| Frontend Testing  | Vitest, React Testing Library   |
| E2E Testing       | Playwright                      |
| API Documentation | OpenAPI                         |
| Containerization  | Docker Compose (local database) |

## 3. Architectural Principles

1. The backend is authoritative for game rules and state.
2. The frontend must not calculate authoritative scores.
3. Game engines must be independent of their content providers.
4. External API calls must not be required during gameplay.
5. All active game sessions must be recoverable.
6. Features must be modular and independently testable.
7. Avoid premature abstractions and unnecessary infrastructure.
8. Authentication and real-time multiplayer are outside MVP scope.

## 4. System Architecture

```text
React SPA
  |
  | HTTPS / REST
  v
ASP.NET Core Web API
  |
  +-- Players
  +-- Games
  +-- Hangman
  +-- PokemonCatalog
  |
  +-- EF Core
        |
        v
     PostgreSQL

PokéAPI
  |
  +-- Catalog Import / Seeding
        |
        v
     PostgreSQL
```

## 5. Repository Structure

```text
pokemon-games/
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   ├── features/
│   │   │   ├── players/
│   │   │   ├── games/
│   │   │   └── hangman/
│   │   └── shared/
│   └── package.json
│
├── backend/
│   ├── PokemonGames.Api/
│   ├── PokemonGames.Application/
│   ├── PokemonGames.Domain/
│   └── PokemonGames.Infrastructure/
│
├── tests/
│   ├── PokemonGames.UnitTests/
│   └── PokemonGames.IntegrationTests/
│
├── docs/
│   ├── architecture.md
│   ├── conventions.md
│   └── backlog.md
│
├── docker-compose.yml
└── README.md
```

### Backend Dependencies

```text
Api -> Application
Api -> Infrastructure
Application -> Domain
Infrastructure -> Application
Infrastructure -> Domain
Domain -> No project dependencies
```

The Domain project must not depend on EF Core, ASP.NET Core, or external services.

## 6. Functional Requirements

### 6.1 Game Modes

| Mode              | Players | Description                            |
| ----------------- | ------- | -------------------------------------- |
| Single-player     | 1       | One player completes five rounds       |
| Local multiplayer | 2–4     | Players alternate rounds on one device |

### 6.2 Turn Order

Each player completes one round before the next cycle begins.

Example with three players:

```text
Player A — Round 1
Player B — Round 1
Player C — Round 1

Player A — Round 2
Player B — Round 2
Player C — Round 2

...

Player C — Round 5
```

### 6.3 Hangman Rules

| Rule                      | Value                             |
| ------------------------- | --------------------------------- |
| Rounds per player         | 5                                 |
| Maximum incorrect guesses | 6                                 |
| Correct guess             | Reveals all matching positions    |
| Incorrect guess           | Consumes one attempt              |
| Repeated guess            | Rejected without penalty          |
| Hint                      | Pokémon type(s)                   |
| Timer                     | None                              |
| Correctly solved Pokémon  | 100 - (10 × incorrect guesses)    |
| Failed Pokémon            | 0                                 |
| Pokémon repetition        | Not allowed within a game session |

A round is won when all required letters are revealed.

A round is lost after six incorrect guesses.

The Pokémon name and image become visible when the round ends.

### 6.4 Word Normalization

The game uses canonical English Pokémon names.

Names must be normalized consistently for gameplay.

- Letter comparisons are case-insensitive.
- Guessable characters are A–Z.
- Non-letter characters are displayed automatically.
- Non-letter characters do not require guesses.
- The original display name is preserved.

The normalization logic must be centralized and unit-tested.

### 6.5 Scoring and Results

Each completed round stores its score.

A participant's total score is the sum of their five round scores.

Participants are ranked by total score.

Players with equal total scores share the same ranking.

The game ends when all participants have completed five rounds.

## 7. Domain Model

### 7.1 Player

| Property     | Type     |
| ------------ | -------- |
| Id           | Guid     |
| OwnerId      | Guid     |
| DisplayName  | string   |
| CreatedAtUtc | DateTime |

Players are persistent and reusable within the same anonymous browser installation.

### 7.2 GameSession

| Property         | Type       |
| ---------------- | ---------- |
| Id               | Guid       |
| OwnerId          | Guid       |
| GameType         | string     |
| Status           | GameStatus |
| TotalRounds      | int        |
| CurrentTurnIndex | int        |
| CreatedAtUtc     | DateTime   |
| CompletedAtUtc   | DateTime?  |

GameStatus:

```text
InProgress
Completed
Abandoned
```

### 7.3 GameParticipant

| Property      | Type |
| ------------- | ---- |
| Id            | Guid |
| GameSessionId | Guid |
| PlayerId      | Guid |
| TurnOrder     | int  |
| TotalScore    | int  |

### 7.4 HangmanRound

| Property          | Type        |
| ----------------- | ----------- |
| Id                | Guid        |
| GameSessionId     | Guid        |
| GameParticipantId | Guid        |
| PokemonId         | int         |
| RoundNumber       | int         |
| Status            | RoundStatus |
| RemainingAttempts | int         |
| Score             | int         |

RoundStatus:

```text
InProgress
Won
Lost
```

### 7.5 HangmanGuess

| Property       | Type     |
| -------------- | -------- |
| Id             | Guid     |
| HangmanRoundId | Guid     |
| Letter         | char     |
| IsCorrect      | bool     |
| CreatedAtUtc   | DateTime |

### 7.6 Pokemon

| Property   | Type                  |
| ---------- | --------------------- |
| Id         | int                   |
| Name       | string                |
| Types      | Collection of strings |
| SpriteUrl  | string                |
| Generation | int                   |

The MVP catalog contains Pokémon with IDs 1–150.

## 8. Game Lifecycle

### 8.1 Create Game

1. Validate that 1–4 distinct players were selected.
2. Verify that all players belong to the current anonymous owner.
3. Create the GameSession.
4. Create GameParticipant records.
5. Initialize the first HangmanRound.
6. Select a random Pokémon not previously used in the session.
7. Persist the game state.

### 8.2 Guess Letter

1. Validate game ownership.
2. Validate that the game is active.
3. Validate that the round is active.
4. Validate that the requested letter is A–Z.
5. Reject repeated guesses without penalty.
6. Check the letter against the normalized Pokémon name.
7. Persist the guess.
8. Update remaining attempts.
9. Determine whether the round was won or lost.
10. Calculate the score when the round ends.
11. Return the updated public game state.

### 8.3 Next Turn

1. Validate that the current round is finished.
2. Determine the next participant and round number.
3. If all rounds are completed, finalize the game.
4. Otherwise, select an unused Pokémon.
5. Create the next HangmanRound.
6. Persist and return the new state.

### 8.4 Resume Game

1. Load the existing GameSession.
2. Verify ownership.
3. Load participants, rounds, and guesses.
4. Reconstruct the public game state.
5. Continue from the current turn.

The backend must never expose the secret Pokémon name, ID, image URL, or other answer-revealing fields while its round is active.

## 9. API Contract

Base path:

```text
/api
```

### Players

| Method | Endpoint      | Description        |
| ------ | ------------- | ------------------ |
| POST   | /players      | Create player      |
| GET    | /players      | List saved players |
| PATCH  | /players/{id} | Rename player      |

### Games

| Method | Endpoint              | Description    |
| ------ | --------------------- | -------------- |
| POST   | /games                | Create game    |
| GET    | /games                | List games     |
| GET    | /games/{id}           | Get game state |
| POST   | /games/{id}/start     | Start game     |
| POST   | /games/{id}/guesses   | Submit letter  |
| POST   | /games/{id}/next-turn | Advance turn   |
| POST   | /games/{id}/abandon   | Abandon game   |
| GET    | /games/{id}/results   | Get results    |

### Example: Create Game

```json
{
  "gameType": "Hangman",
  "playerIds": [
    "11111111-1111-1111-1111-111111111111",
    "22222222-2222-2222-2222-222222222222"
  ]
}
```

### Example: Submit Guess

```json
{
  "letter": "A"
}
```

### Example: Public Round State

```json
{
  "roundId": "33333333-3333-3333-3333-333333333333",
  "roundNumber": 2,
  "maskedWord": "P _ K _ C H U",
  "remainingAttempts": 4,
  "guessedLetters": ["P", "K", "C", "H", "U", "E", "O"],
  "pokemonTypes": ["Electric"],
  "status": "InProgress",
  "score": null
}
```

The actual API responses may include additional safe metadata.

## 10. Anonymous Identity

Authentication is outside the MVP.

The application must still isolate players and games between anonymous browser installations.

Recommended approach:

1. The backend issues an opaque, cryptographically random installation credential.
2. The browser stores it in a Secure, HttpOnly, SameSite cookie.
3. The backend resolves the installation identity from the credential.
4. Players and games are scoped to that identity.
5. The frontend never supplies a trusted OwnerId.

A public installation identifier alone is not sufficient authorization.

This model does not provide account recovery or cross-device synchronization.

## 11. Data Persistence

PostgreSQL stores:

- Players
- Game sessions
- Participants
- Hangman rounds
- Guesses
- Pokémon catalog
- Anonymous installation credentials or their secure hashes

EF Core migrations manage schema changes.

Database operations that advance game state must be transactional.

Concurrent requests must not advance the same turn twice or award duplicate points.

Use optimistic concurrency control or an equivalent database-safe mechanism.

## 12. Pokémon Catalog

PokéAPI is used only for catalog import and maintenance.

Source endpoints:

```text
GET https://pokeapi.co/api/v2/pokemon?limit=150&offset=0
GET https://pokeapi.co/api/v2/pokemon/{id}
```

Import:

- ID
- Canonical English name
- Types
- Sprite URL
- Generation

The import must be idempotent.

Missing or invalid Pokémon records must be detected and reported.

The game must work without PokéAPI connectivity after successful import.

## 13. Frontend State

### Zustand

Use for temporary UI state:

- Local UI preferences
- Dialog visibility
- Non-authoritative interaction state

### TanStack Query

Use for server state:

- Players
- Game sessions
- Active game
- Current round
- Results

Do not duplicate authoritative game state in Zustand.

## 14. Error Handling

Use ProblemDetails for API errors.

Expected cases include:

- Invalid player selection
- Unauthorized anonymous owner
- Game not found
- Game already completed
- Invalid letter
- Repeated letter
- Round already finished
- Invalid turn transition
- Catalog unavailable or incomplete

Errors must not expose internal implementation details or secret Pokémon information.

## 15. Testing Strategy

### Unit Tests

- Word normalization
- Correct and incorrect guesses
- Repeated guesses
- Round completion
- Scoring
- Turn order
- Pokémon uniqueness
- Ranking and ties

### Integration Tests

- Player persistence
- Game creation
- Guess submission
- Ownership isolation
- Concurrent guess handling
- Game recovery
- Session completion

### Frontend Tests

- Game setup
- Keyboard interactions
- Hangman rendering
- Turn transition
- Results screen

### E2E Tests

- Complete single-player game
- Complete local multiplayer game
- Resume interrupted game

## 16. Future Extensions

Not included in the MVP:

- Authenticated accounts
- SignalR multiplayer
- Online rooms
- Matchmaking
- Global leaderboards
- Timed challenges
- Additional games
- Additional Pokémon generations

Future multiplayer must reuse the existing backend game engine rather than duplicating game rules in the client.

## 17. MVP Definition of Done

The MVP is complete when:

1. Users can create and reuse players.
2. Users can start single-player and local multiplayer games.
3. Every player completes five rounds.
4. All Pokémon are selected from the first 150.
5. No Pokémon repeats within a game session.
6. Guesses, attempts, turns, and scores are validated by the backend.
7. Players can resume interrupted games.
8. Results and history are persisted.
9. The application works on desktop and mobile.
10. Critical unit, integration, and E2E tests pass.
11. The application is deployed and accessible.
