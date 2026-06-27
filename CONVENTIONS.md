# Conventions

The rules for working in this starter. Most are **lint-enforced** (you'll get an
error); the rest are judgment calls a reviewer/agent must uphold. The authoritative
Phaser knowledge lives in the official Phaser skills at
`node_modules/phaser/skills/<topic>/SKILL.md` — read those for any API question.

## Architecture (enforced)

- **All game rules live in `src/logic/**`, with zero `phaser` imports.** Pure data
  in, data out. Scenes/objects/services are thin adapters. (`no-restricted-imports`)
- **One scene class per file** under `src/scenes/`; flow is Boot → Preloader →
  MainMenu → Game → GameOver.
- **Typed keys only.** Import scene/event/asset keys from `consts.ts` / `assets.ts`;
  never inline raw strings.

## Determinism (enforced)

- **No `Math.random()` / `Date.now()` / `performance.now()`** in game code. Use the
  seeded `Rng` (`src/logic/rng.ts`) and the scene delta. This keeps replays and the
  verification suite reproducible.

## Performance (enforced)

- **No object/tween/timer creation in `update()`** and **no per-frame allocation**
  (object/array literals, `.map`/`.filter`/`.reduce`). Create in `create()`, pool,
  and reuse. (`phaser/no-create-in-update`, `phaser/no-allocation-in-update`)
- **The zero-allocation indexed `for (let i = …)` is allowed** in hot paths (it's
  the one place `let` is permitted).
- **Pool transient objects** (items, bullets, particles) — never `new`/`destroy`
  per spawn.

## State, events, lifecycle (enforced)

- **Global state via the registry** (typed); **events via the single EventsCenter**
  (`src/services/events.ts`) — never `game.events`.
- **Persist only through `SaveService`** — no raw `localStorage` outside
  `src/services/**`. Bump the schema version + extend the migration chain on change.
- **Every persistent external listener** (`this.input` / `scale` / `window`) must be
  cleaned up on scene shutdown (`require-shutdown-cleanup`). Prefer `once` for
  one-shots.

## Assets

- Placeholder art is generated programmatically in the Preloader (zero binary
  assets). When adding real art, run it through the asset pipeline
  (`free-tex-packer-core` / `audiosprite` / BMFont) and generate `src/assets.ts`
  — see the `phaser-asset-pipeline` skill.

## Verification (the definition of done)

- **Nothing is done until an agent has played the running game and confirmed the
  change** against its acceptance criteria, with a committed evidence artifact, and
  a `tests/e2e` verification spec that re-runs in CI. Verification **is** UAT.
- Drive the canvas through `window.__VERIFY__` (enabled with `?uat=1`): seed the
  RNG, read state, inject input. Pure game rules are unit-tested in `tests/logic`;
  scenes/services are verified end-to-end in `tests/e2e`.

## Judgment calls (not lint-enforced)

- Choosing the right pooling strategy / when to reach for `SpriteGPULayer`.
- Game feel, juice, and art direction.
- Whether a change is genuinely non-behavioral enough to be `verification-exempt`
  (use the label sparingly and it is logged, never silent).
