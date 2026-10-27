# Repository guidance

## Current state and sources

- This repository is documentation-only: the application layout in `docs/architecture.md` is planned, not implemented. There are no manifests, lockfiles, build/test scripts, CI workflows, or runnable services yet; do not assume setup or verification commands exist.
- Before implementation, read `docs/architecture.md` (design and business rules), `docs/conventions.md` (implementation and agent workflow), and the relevant item in `docs/backlog.md` (dependencies and acceptance criteria).
- For backlog work, verify prerequisites before starting dependent tasks, update the task status, and mark it Done only after its acceptance criteria are satisfied.

## Planned implementation boundaries

- Target stack: React/TypeScript/Vite frontend; ASP.NET Core on .NET 10 with EF Core/PostgreSQL backend. The planned directories are `frontend/`, `backend/`, and `tests/`; see the architecture document before scaffolding.
- Backend project dependencies: `Api -> Application + Infrastructure`, `Infrastructure -> Application + Domain`, `Application -> Domain`; `Domain` has no project dependencies and must not reference EF Core, ASP.NET Core, or external services.
- Keep game rules in the backend, independent of PokéAPI and UI. TanStack Query owns server state; Zustand is for client UI state, not a duplicate of authoritative game state.
- PokéAPI is for idempotent catalog import only. Gameplay must work offline from PokéAPI after import; the MVP uses IDs 1–150, not all 151 Generation I Pokémon.
- Multiplayer means 2–4 players sharing one device. Accounts, online multiplayer/SignalR, and additional games or generations are outside the MVP unless explicitly requested.

## Easy-to-miss requirements

- No accounts does not mean shared data: scope players and games to an anonymous browser installation; never trust a frontend-supplied `OwnerId`. The documented approach is a backend-issued opaque credential in a Secure, HttpOnly, SameSite cookie.
- Active-round responses must hide the Pokémon name, ID, sprite URL, and any other answer-revealing fields; type hints are allowed. Reveal the answer only after the round ends.
- Each player gets five rounds, rotating players after each round. Pokémon cannot repeat anywhere in the session. Repeated guesses are rejected without consuming attempts; consult the documented scoring and normalization rules before changing the engine.
- Persist recoverable sessions and make game-state transitions transactional and concurrency-safe: retries or concurrent requests must not duplicate scores or advance a turn twice.
- Use EF Core migrations, not `EnsureCreated` for normal startup; do not edit applied migrations.

## Language and task reporting

- Code, documentation, and Conventional Commit messages are in English; user-facing application text is initially Spanish. Pokémon names remain canonical English names.
- Planned test tools are xUnit, Vitest/React Testing Library, and Playwright; none is configured yet. `docs/conventions.md` requires tests for new business rules and regression coverage for fixes when practical.
- After implementation tasks, use the sections `Implemented`, `Files changed`, `Tests`, and `Pending` as specified in `docs/conventions.md`; identify unavailable checks or blockers explicitly.
