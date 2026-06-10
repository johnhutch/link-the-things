# Progress

**Last updated:** 2026-06-10
**Active branch:** main

Current state + a rolling shipped-log. Planned/not-started work lives in `TODOS.md`; the *why* behind decisions lives in `DECISIONS.md`.

---

## Current focus

The brutalist design now covers the whole site. Phase 2 (authoring) and the core
of Phase 3 (public play page + our Stimulus game loop, stats, share cube) are in
place. The open frontier is the **auth & accounts epic** — no-login creation,
claim-on-signup, creator dashboards — gated by open decisions **D1–D4** in
`TODOS.md`, which must be worked via the `grill-me` skill before any code. Deploy
is decided (ADR-0004) but not yet run end-to-end (waits on one-time NAS setup).

## Shipped log (most recent first)

- **Homepage hero + create stickers** — added a grid-breaking `Create ↗` sticker
  (`.m-create-sticker`) to the top-right of the play page and homepage hero,
  linking to the auth-gated `new_puzzle_path`. The homepage `NOTimes` became a
  white textura-blackletter nameplate (self-hosted UnifrakturMaguntia, OFL)
  layered over the `QUARTETS` wordmark; the create sticker floats absolute so it
  shares the wordmark's plane instead of pushing it down. `body.theme-brutal`
  got `overflow-x: hidden` to clip the intentional edge-bleed on phones.
- **Brutalist theme is now site-wide** — promoted `theme-brutal` from an opt-in
  body class (homepage + styleguide only) to the layout default; a page opts out
  by setting its own `content_for(:body_class)`. Extended `_brutal.scss` to cover
  the previously-unthemed modules: puzzle lists (browse + dashboard), stats
  panels, the author form + category fieldsets, draft badge, flash messages, and
  interior page headings. Bare `button_to`/`f.submit` buttons picked up the
  `.m-btn`/`--pop` vocabulary; interior `<h1>`s now run through `multicolor`. One
  system-spec assertion loosened to a case-insensitive match since the dashboard
  title is now display-uppercased (CSS, but Selenium reads rendered text). All 45
  specs green.
- **Multicolor headers re-roll on every load** — dropped `Multicolor`'s MD5
  seed so colors *and* break positions re-randomize per call (run length now
  3–6). Kills the "frozen purple" look where deterministic seeding pinned a
  phrase's banding forever. Server-side, zero JS; the contract is headers stay
  out of `<% cache %>` blocks (an optional `seed:` pins a banding for any future
  must-cache header). Spec flipped from determinism → re-roll + seed.
- **Auto-save drafts** — debounced Stimulus controller (`autosave_controller.js`):
  first edit POSTs to mint the draft, then flips the form to PATCH it. Endpoint
  answers quietly (201 + `Location`, then 204). `Puzzle#title` is now publish-only
  so untitled partial drafts persist. Covered by `puzzle_autosave_spec.rb`.
- **System-spec harness** — headless Chrome at a phone viewport
  (`spec/support/capybara.rb`), guarding against chromedriver/Chrome version
  drift; auto-save resilience + author/publish system specs. Also cleaned a
  malformed global `~/.bundle/config` that was breaking `bundle exec`.

## Known not-done / watch-outs

- The author→publish system spec still stops at the dashboard; extending it
  through to the (now-existing) share URL is a TODO quick-win.
- `docs/PLAN.md`'s schema sketch calls `Group#words` a "PG array"; it's actually
  a **jsonb** column. Treat jsonb as the truth.
