# Product Backlog — Pokémon Games MVP

## 1. Product Goal

Build a web-based minigame platform featuring Pokémon Hangman as its first game.

The MVP supports:

- First 150 Pokémon from PokéAPI.
- Single-player games.
- Local multiplayer with 2–4 players.
- Five rounds per player.
- Persistent, reusable players.
- Recoverable game sessions.
- Pokémon type hints.
- Persistent scores and game history.
- Responsive desktop and mobile interfaces.

Online multiplayer, authentication, and additional games are outside MVP scope.

## 2. Priorities

| Priority | Meaning               |
| -------- | --------------------- |
| P0       | Required for MVP      |
| P1       | Important improvement |
| P2       | Future enhancement    |

## 3. Status Definitions

| Status      | Meaning                            |
| ----------- | ---------------------------------- |
| Todo        | Not started                        |
| In Progress | Currently being implemented        |
| Blocked     | Cannot proceed                     |
| Done        | Implemented, tested, and validated |

## 4. Epic Overview

| Epic  | Name                        | Priority | Status |
| ----- | --------------------------- | -------- | ------ |
| EP-01 | Project Foundation          | P0       | In Progress |
| EP-02 | Pokémon Catalog             | P0       | Todo   |
| EP-03 | Player Management           | P0       | Todo   |
| EP-04 | Hangman Engine              | P0       | Todo   |
| EP-05 | Game Sessions & Multiplayer | P0       | Todo   |
| EP-06 | Quality & Deployment        | P0       | Todo   |

---

## EP-01 — Project Foundation

**Goal:** Establish the frontend, backend, database, and development environment.

| ID     | Task                                             | Priority | Depends On     | Status |
| ------ | ------------------------------------------------ | -------- | -------------- | ------ |
| FND-01 | Initialize monorepo structure                    | P0       | —              | Done   |
| FND-02 | Create .NET 10 solution and projects             | P0       | FND-01         | Done   |
| FND-03 | Create React + TypeScript + Vite app             | P0       | FND-01         | Done   |
| FND-04 | Configure Tailwind CSS and routing               | P0       | FND-03         | Todo   |
| FND-05 | Configure PostgreSQL and EF Core                 | P0       | FND-02         | Todo   |
| FND-06 | Configure Docker Compose                         | P0       | FND-05         | Todo   |
| FND-07 | Configure OpenAPI and ProblemDetails             | P0       | FND-02         | Todo   |
| FND-08 | Configure frontend API client and TanStack Query | P0       | FND-03         | Todo   |
| FND-09 | Implement anonymous installation identity        | P0       | FND-05         | Todo   |
| FND-10 | Configure testing projects                       | P0       | FND-02, FND-03 | Todo   |

### Acceptance Criteria

- Backend builds successfully.
- Frontend builds successfully.
- PostgreSQL runs locally.
- EF Core migrations can be applied.
- API health endpoint responds successfully.
- Frontend can communicate with the backend.
- Anonymous installation credentials are securely issued and recognized.
- Unit and frontend test runners work.
- README contains local setup instructions.

---

## EP-02 — Pokémon Catalog

**Goal:** Import and persist the first 150 Pokémon.

| ID     | Task                                       | Priority | Depends On | Status |
| ------ | ------------------------------------------ | -------- | ---------- | ------ |
| PKM-01 | Create Pokemon entity and EF configuration | P0       | FND-05     | Todo   |
| PKM-02 | Implement PokéAPI client                   | P0       | PKM-01     | Todo   |
| PKM-03 | Implement catalog import service           | P0       | PKM-02     | Todo   |
| PKM-04 | Persist Pokémon names, types, and sprites  | P0       | PKM-03     | Todo   |
| PKM-05 | Implement idempotent catalog seeding       | P0       | PKM-04     | Todo   |
| PKM-06 | Add catalog validation tests               | P0       | PKM-05     | Todo   |

### Acceptance Criteria

- Exactly 150 Pokémon are available.
- Pokémon IDs range from 1 to 150.
- Each Pokémon has a valid name.
- Each Pokémon has at least one type.
- Sprite URLs are stored when available.
- Repeated imports do not create duplicates.
- Missing or invalid records are detected.
- Gameplay does not require PokéAPI connectivity.

---

## EP-03 — Player Management

**Goal:** Allow users to create and reuse local players.

| ID     | Task                                      | Priority | Depends On     | Status |
| ------ | ----------------------------------------- | -------- | -------------- | ------ |
| PLY-01 | Create Player entity and migration        | P0       | FND-09         | Todo   |
| PLY-02 | Implement player creation endpoint        | P0       | PLY-01         | Todo   |
| PLY-03 | Implement player listing endpoint         | P0       | PLY-02         | Todo   |
| PLY-04 | Implement player rename endpoint          | P0       | PLY-02         | Todo   |
| PLY-05 | Build player management UI                | P0       | PLY-03, FND-08 | Todo   |
| PLY-06 | Build player selection UI                 | P0       | PLY-05         | Todo   |
| PLY-07 | Add player validation and ownership tests | P0       | PLY-04         | Todo   |

### Acceptance Criteria

- Users can create players using display names.
- Players persist after browser refresh.
- Existing players can be selected for new games.
- Player names can be updated.
- A game accepts 1–4 distinct players.
- Players from another installation cannot be accessed or modified.
- Invalid names are rejected.

---

## EP-04 — Hangman Engine

**Goal:** Implement the core game rules independently of the UI and PokéAPI.

| ID     | Task                                      | Priority | Depends On    | Status |
| ------ | ----------------------------------------- | -------- | ------------- | ------ |
| HNG-01 | Implement word normalization              | P0       | FND-02        | Todo   |
| HNG-02 | Implement guess evaluation                | P0       | HNG-01        | Todo   |
| HNG-03 | Implement attempt tracking                | P0       | HNG-02        | Todo   |
| HNG-04 | Implement win/loss detection              | P0       | HNG-03        | Todo   |
| HNG-05 | Implement scoring                         | P0       | HNG-04        | Todo   |
| HNG-06 | Implement random unused Pokémon selection | P0       | PKM-06        | Todo   |
| HNG-07 | Implement Pokémon type hints              | P0       | PKM-04        | Todo   |
| HNG-08 | Add comprehensive engine unit tests       | P0       | HNG-01–HNG-07 | Todo   |

### Acceptance Criteria

- Words are compared case-insensitively.
- Non-letter characters are handled automatically.
- Correct guesses reveal all occurrences.
- Incorrect guesses reduce remaining attempts by one.
- Duplicate guesses do not consume attempts.
- Six incorrect guesses end the round.
- A completed word ends the round successfully.
- Winning score equals 100 minus 10 per incorrect guess.
- Losing score is zero.
- Pokémon types are available as hints.
- The engine does not depend on EF Core, HTTP, or React.
- Unit tests cover all important rules and edge cases.

---

## EP-05 — Game Sessions & Multiplayer

**Goal:** Implement complete playable sessions, local multiplayer, persistence, and results.

| ID     | Task                                  | Priority | Depends On             | Status |
| ------ | ------------------------------------- | -------- | ---------------------- | ------ |
| GAM-01 | Create game session domain entities   | P0       | FND-05                 | Todo   |
| GAM-02 | Implement game creation               | P0       | GAM-01, PLY-07         | Todo   |
| GAM-03 | Implement game start and first round  | P0       | GAM-02, HNG-08         | Todo   |
| GAM-04 | Implement guess submission endpoint   | P0       | GAM-03                 | Todo   |
| GAM-05 | Implement game state endpoint         | P0       | GAM-04                 | Todo   |
| GAM-06 | Build Hangman board and keyboard      | P0       | GAM-05                 | Todo   |
| GAM-07 | Build single-player gameplay flow     | P0       | GAM-06                 | Todo   |
| GAM-08 | Implement turn rotation               | P0       | GAM-04                 | Todo   |
| GAM-09 | Implement next-turn endpoint          | P0       | GAM-08                 | Todo   |
| GAM-10 | Build local multiplayer flow          | P0       | GAM-07, GAM-09         | Todo   |
| GAM-11 | Implement final scores and rankings   | P0       | GAM-10                 | Todo   |
| GAM-12 | Build results screen                  | P0       | GAM-11                 | Todo   |
| GAM-13 | Implement game recovery               | P0       | GAM-05                 | Todo   |
| GAM-14 | Implement game history                | P0       | GAM-11                 | Todo   |
| GAM-15 | Implement game abandonment            | P0       | GAM-05                 | Todo   |
| GAM-16 | Add concurrency and integration tests | P0       | GAM-04, GAM-09, GAM-13 | Todo   |

### Acceptance Criteria

#### Game Creation

- A game accepts 1–4 distinct players.
- Each participant receives five rounds.
- The server controls the participant order.
- Pokémon are not repeated within a game.

#### Gameplay

- The frontend displays the masked word.
- Pokémon types are visible as hints.
- Players submit letters using the interface.
- The server validates each guess.
- Remaining attempts and the drawing update correctly.
- The secret Pokémon is not exposed during an active round.
- The Pokémon name and image are revealed after the round.

#### Multiplayer

- Players alternate after each completed round.
- The current player is clearly identified.
- The device can be passed to the next player.
- The next round cannot start before the current one ends.
- All participants complete five rounds.
- Total scores are calculated by the backend.

#### Recovery

- Active sessions persist in PostgreSQL.
- Reloading the page does not reset the game.
- A user can reopen an unfinished game.
- Previously submitted guesses remain visible.
- The correct player and round are restored.

#### Results

- Final scores are displayed.
- Players are ranked by total score.
- Ties are handled consistently.
- Completed games appear in history.

#### Concurrency

- Duplicate requests do not award points twice.
- A turn cannot advance more than once.
- Concurrent guesses cannot corrupt round state.

---

## EP-06 — Quality & Deployment

**Goal:** Validate, polish, and publish the MVP.

| ID     | Task                                 | Priority | Depends On     | Status |
| ------ | ------------------------------------ | -------- | -------------- | ------ |
| QLT-01 | Implement responsive layouts         | P0       | GAM-06         | Todo   |
| QLT-02 | Add loading, error, and empty states | P0       | GAM-10         | Todo   |
| QLT-03 | Add accessibility improvements       | P0       | QLT-01         | Todo   |
| QLT-04 | Add backend integration tests        | P0       | GAM-16         | Todo   |
| QLT-05 | Add frontend component tests         | P0       | GAM-12         | Todo   |
| QLT-06 | Add Playwright E2E tests             | P0       | GAM-13         | Todo   |
| QLT-07 | Configure CI workflows               | P0       | QLT-04, QLT-05 | Todo   |
| QLT-08 | Configure production environment     | P0       | QLT-07         | Todo   |
| QLT-09 | Deploy frontend and backend          | P0       | QLT-08         | Todo   |
| QLT-10 | Complete documentation               | P0       | QLT-09         | Todo   |

### Acceptance Criteria

- UI works on desktop and mobile.
- Touch controls are usable.
- Loading and error states are displayed.
- Keyboard navigation is supported.
- Critical unit tests pass.
- Integration tests pass.
- E2E tests cover single-player, multiplayer, and recovery.
- CI validates frontend and backend.
- Production secrets are securely configured.
- The application is publicly accessible.
- README documents setup, architecture, and deployment.

---

## 5. Suggested Implementation Milestones

| Milestone | Scope                 | Expected Result                           |
| --------- | --------------------- | ----------------------------------------- |
| M1        | EP-01                 | Running frontend, backend, and database   |
| M2        | EP-02 + EP-03         | Pokémon catalog and persistent players    |
| M3        | EP-04 + GAM-01–GAM-07 | Playable single-player game               |
| M4        | GAM-08–GAM-12         | Local multiplayer and final results       |
| M5        | GAM-13–GAM-16         | Recovery, history, and concurrency safety |
| M6        | EP-06                 | Tested and deployed MVP                   |

## 6. Out of Scope

The following features must not be implemented during the MVP unless explicitly requested:

| Feature                        | Target |
| ------------------------------ | ------ |
| Authentication and accounts    | Future |
| Online multiplayer             | Future |
| SignalR                        | Future |
| Matchmaking                    | Future |
| Global leaderboards            | Future |
| Chat                           | Future |
| Additional games               | Future |
| Additional Pokémon generations | Future |
| Timed challenges               | Future |
| Player avatars and cosmetics   | Future |

## 7. Implementation Workflow

For each task:

1. Review architecture and conventions.
2. Verify dependencies are completed.
3. Implement the smallest functional change.
4. Add or update relevant tests.
5. Run available validation commands.
6. Update the task status.
7. Report implementation details and remaining issues.

Do not begin dependent tasks when prerequisites are incomplete.

Do not mark tasks as Done without satisfying their acceptance criteria.

## 8. MVP Completion Checklist

- [ ] Frontend and backend run locally.
- [ ] PostgreSQL persistence works.
- [ ] First 150 Pokémon are imported.
- [ ] Players can be created and reused.
- [ ] Single-player mode works.
- [ ] Local multiplayer supports 2–4 players.
- [ ] Every player completes five rounds.
- [ ] Pokémon type hints work.
- [ ] Scoring rules are enforced.
- [ ] Pokémon do not repeat within a game.
- [ ] Active games can be resumed.
- [ ] Results and history persist.
- [ ] Secret answers remain server-side.
- [ ] Critical automated tests pass.
- [ ] Responsive UI works.
- [ ] Application is deployed.
