# Build Plan — Link the Things

The gameplan, phased so each step ships something real and testable. We're
going TDD, so "build X" always means "spec X, then build X."

**Status key:** ✅ done · 🚧 in progress · ⬜ not started

---

## Phase 0 — Foundation ✅

- ✅ `rails new` with PostgreSQL + Sass (dartsass-rails, Propshaft)
- ✅ SMACSS stylesheet structure (`_variables`, `_base`, `_modules`, `_state` +
  manifest), compiling clean
- ✅ Project docs (README, CLAUDE.md, this plan)
- ⬜ RSpec + Capybara installed and configured (`rails_helper`, system specs,
  factory_bot, a green `bin/rspec`)
- ⬜ Render config (`render.yaml` or dashboard service) — wire up auto-deploy
  from `main` early so we never have a "works locally only" surprise

## Phase 1 — Data model + auth ⬜

- ⬜ Devise, superuser only (no public sign-up; seed the one admin)
- ⬜ Models: `Puzzle` (title, author, published/draft state) → `Group`
  (category description, color enum: blue/green/yellow/purple) → has many words.
  Decide words as a column/array vs. a `Card` model — start simple.
- ⬜ Validations: exactly 4 groups, exactly 4 words each, colors unique per
  puzzle. These are the rules the form and importer both lean on.

## Phase 2 — Authoring ⬜

- ⬜ Creation form: four color-coded `m-group` blocks, Answers + Description per
  group, Title + Author at the bottom (mirror swellgarfo's field order so muscle
  memory carries over)
- ⬜ **Auto-save drafts** — debounced Turbo/Stimulus save on change. The
  non-negotiable feature. A draft is a `Puzzle` in draft state.
- ⬜ Superuser dashboard: list of published puzzles + drafts (with `is-draft`
  badge), edit/continue/delete
- ⬜ Publish action (draft → published)

## Phase 3 — Play ⬜

- ⬜ Pick + embed the open-source vanilla-JS Connections engine (evaluate
  maintenance, license, ease of feeding it our JSON). No React.
- ⬜ Public puzzle page: feed the engine a puzzle's data, full game loop
  (select 4 → submit → reveal/mistake → win/lose)
- ⬜ Public puzzle index — browsable list (this is the Q11 B→ "anyone on the
  internet" decision)
- ⬜ Anonymous player identity: cookie/session token persisted for stats

## Phase 4 — Stats + sharing ⬜

- ⬜ Record attempts: per play — mistakes, guess sequence, solved?
- ⬜ Per-puzzle stats view: total attempts, solve rate, mistakes-per-attempt,
  common wrong guesses
- ⬜ Emoji result cube (🟨🟩🟦🟪) generated from the attempt's guess rows,
  copy-to-share

## Phase 5 — Import + polish ⬜

- ⬜ `puzzles:import_obsidian` rake task — parse the existing `.md` archive
  (formats are inconsistent across the 8 puzzles; normalize on the way in) and
  seed the DB
- ⬜ JSON export per puzzle
- ⬜ Mobile pass — iPhone is the primary device; the whole thing has to feel
  right on a phone
- ⬜ Production deploy on Render, real smoke test

---

## Open questions / decisions deferred

- Words storage: array column on `Group` vs. a `Card` model. Leaning array for
  v1; revisit if stats need per-card identity.
- Which open-source Connections engine specifically — pick during Phase 3.
- Whether common-wrong-guesses needs its own table or can be derived from stored
  attempt guess-sequences. Probably derivable.
