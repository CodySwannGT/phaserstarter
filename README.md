# Phaser 4 Starter — Collector

An opinionated, **agent-maintained** Phaser 4 + TypeScript + Vite starter, governed
by [Lisa](https://github.com/CodySwannGT/lisa). Every best practice is enforced by
lint rules, type checks, git hooks, and CI — and nothing is "done" until an agent
has actually **played the game** and confirmed it (the verification/UAT suite).

The included game ("Collector" — slide to catch falling items) is a vertical slice
whose job is to exercise every enforced pattern: a pure deterministic simulation,
pooled sprites, semantic input, save/persistence, audio, scene flow, and the
verification bridge.

## Quick start

```bash
bun install
bun run dev        # http://localhost:5173  (arrows / A,D / tap to move; Space to start)
```

## Scripts

| Script | What it does |
| --- | --- |
| `bun run dev` / `build` / `preview` | Vite dev server / production build / preview the build |
| `bun run test` | Vitest unit tests (the pure logic core) |
| `bun run test:cov` | Unit tests with coverage thresholds |
| `bun run test:e2e` | **Playwright verification (UAT)** — builds, serves, and plays the game |
| `bun run typecheck` | `tsc --noEmit` (strict) |
| `bun run lint` / `lint:fix` | oxlint + ESLint (incl. the Phaser rules) |
| `bun run format` | Prettier |
| `bun run knip:check` | Dead-code detection |
| `bun run size` | Bundle-size budget |

## Architecture

```
src/
  logic/        # PURE, engine-free simulation (NO phaser imports) — unit-tested, deterministic
  scenes/       # thin Phaser adapters: Boot → Preloader → MainMenu → Game → GameOver
  services/     # cross-cutting: events bus, input, sound, save (versioned localStorage)
  uat/bridge.ts # verification bridge (window.__VERIFY__), enabled with ?uat=1
  consts.ts     # typed scene keys, event names, tunables
  assets.ts     # typed asset keys (programmatic placeholders here)
  game/config.ts, main.ts
tests/
  logic/        # Vitest unit tests (pure core)
  e2e/          # Playwright verification (UAT) — proves the game actually plays
```

The cardinal rule: **all game rules live in `src/logic` with zero Phaser imports**
(lint-enforced). Scenes are thin orchestrators that render that state and feed it
input. That is what makes the game testable and deterministic.

## The opinions (and how they're enforced)

- **Phaser 4 only** — removed v3 idioms are lint-banned.
- **Determinism** — no `Math.random` / `Date.now` in game code (seeded RNG); enforced.
- **No allocation/creation in `update()`** — pooled sprites; custom ESLint rules.
- **Pure-logic boundary** — `no phaser` imports under `src/logic`.
- **Typed keys** — no raw string asset/scene/event keys.
- **Storage only via `SaveService`**; **events only via the EventsCenter** (never `game.events`).
- **Strict TypeScript** — `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, etc.

See [CONVENTIONS.md](./CONVENTIONS.md) for the full list and the un-lintable ones.

## Verification (UAT)

`bun run test:e2e` boots the production build and drives the canvas through the
in-game `window.__VERIFY__` bridge — seeding the RNG, injecting input, and reading
state — to confirm real behavior (boot, input, scoring, game-over). A change isn't
done until a verification spec proves it.
