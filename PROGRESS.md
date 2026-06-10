# Progress

**Last updated:** 2026-06-10
**Active branch:** main

Current state + a rolling shipped-log. Planned/not-started work lives in `TODOS.md`; the *why* behind decisions lives in `DECISIONS.md`.

---

## Current focus

Phases 0–4 are **done**, and Phase 5 (import + export) is mostly there. The app
plays end-to-end: author → publish → share link → play → stats → emoji cube. What's
left is the **last mile of Phase 5 — a real-device mobile pass and the first
production deploy on the Synology** (config is committed; waits on one-time NAS
setup). Seeding the single superuser is the last Phase 0 loose end.

## Shipped log (most recent first)

- **Renamed the project to Quartets** — folder, GitHub repo (`johnhutch/quartets`,
  old URL redirects), Rails module (`Quartets`), the dev/test Postgres DBs (renamed
  via `ALTER DATABASE`, data intact — 18 published puzzles), and every doc/config
  identifier. The brand was already half-migrated (layout title + `Multicolor`
  wordmark spec). *(Uncommitted as of this writing — see below.)*
- **Design system (brutalist)** — `_brutal.scss`, Space Grotesk webfonts, a
  `/styleguide` page, and `Multicolor` — the deterministic wordmark colorizer that
  bands headings into the four category colors mid-word. Spec'd. Dropped the
  generated GitHub Actions CI (archive-only repo, no CI/CD).
- **Phase 5 — import + export.** `puzzles:import_obsidian` rake task (`ObsidianArchive`,
  forgiving parser, idempotent: complete 4×4 → published, partial → draft, junk
  skipped). JSON export per puzzle (`PuzzleExport`, spec-pinned schema; gated,
  owner-scoped `/puzzles/:id/export`). Both hard-spec'd.
- **Phase 4 — stats + sharing.** Attempts recorded best-effort via
  `POST /p/:share_token/attempts` (anonymous, `player_token` cookie). Owner-scoped
  `/puzzles/:id/stats` (`PuzzleStats`: attempts, solve rate, mistakes distribution,
  common wrong guesses — all derived from the `guesses` jsonb). `EmojiCube` value
  object + copy-to-clipboard share cube. Unit + request + system specs.
- **Phase 3 — play.** Our own Stimulus `game_controller.js` (no droppable vanilla
  engine exists — ADR-0003): shuffle 16, pick-4 → submit → reveal/mistake loop,
  cap at `Puzzle::MAX_MISTAKES`, emits `game:finished` with the guess log. Public
  `/p/:share_token` page (`PlayController`; drafts/bad tokens 404, no login) +
  browsable `/play` index of published puzzles. Win + loss system specs.
- **Deploy pivot — Render → self-hosted Synology** (ADR-0004). DSM Container
  Manager `docker-compose.yml` (app + one Postgres for Solid cache/queue/cable),
  image built on the Mac (`linux/amd64`) and shipped over SSH via `bin/deploy` —
  no registry, no CI. Runbook in `docs/DEPLOY.md`. *(First deploy pending NAS setup.)*
- **Phase 2 — authoring.** Color-coded form (swellgarfo order, answers-first),
  gated `PuzzlesController`, owner-scoped dashboard, publish action, and
  **auto-save drafts** (debounced `autosave_controller.js`: POST to mint, then
  PATCH; title is publish-only so untitled drafts persist). Request + system specs.
- **Phase 0/1 — foundation.** Rails 8 + Postgres + Sass (SMACSS), RSpec/Capybara/
  factory_bot, headless-Chrome phone-viewport system-spec harness, Devise
  (superuser-only), and the `Puzzle`/`Group`/`Attempt` models + validations.

## Known not-done / watch-outs

- **This session's rename is uncommitted** (clean tree → ~17 files changed). DB
  rename + GitHub repo rename are already applied and live.
- **Mobile pass** (real iPhone) and **first Synology production deploy** + an
  end-to-end smoke test are the remaining Phase 5 ⬜s.
- **Seed the superuser** (env-driven creds) — the last Phase 0 loose end.
- Two TODOS open: extend the author→publish system spec to assert it lands on the
  `/p/:share_token` page (now possible), and tune the 1000ms auto-save debounce on
  a real phone.
- `docs/PLAN.md`'s schema sketch calls `Group#words` a "PG array"; it's actually a
  **jsonb** column. Treat jsonb as the truth.
