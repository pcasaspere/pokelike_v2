# AGENTS.md

Vanilla JS/CSS/HTML Pokémon roguelike. **No build, no package manager, no tests, no linter.** Edit a file, reload the browser.

## Run

Must be served over HTTP (`file://` fails — `data/pokedex.json` fetch):

```bash
python3 -m http.server 8000
# http://127.0.0.1:8000/
```

`index.html` cache-busts CSS/JS with `?v=Date.now()`, so a plain reload picks up edits.

Push to `main` → GitHub Pages via `.github/workflows/static.yml` (uploads the repo as-is).

## Architecture

Classic scripts, one global scope, **load order is load-bearing** (`index.html`):

`data` → `map` → `battle` → `endless` → `ui` → `game` → `rules` → `cloud-save`

- `js/data.js` — TYPE_CHART, MOVE_POOL, gym/E4/items, `GEN_RUN_CONFIG`. Boots `data/pokedex.json` (#1–649) + `data/pokedex-mods.json` era overrides (`applyEraStats()` / `getStatsEra()`). Types stay modern 18-type. Campaign gens 1–5; Tower stages 1–5 use that stage’s era stats; Tot (`runGen === 'all'`) uses modern stats. Numeric IDs use the bundle; **form slugs still fetch PokeAPI**.
- `js/map.js` — branching node map. `NODE_TYPES` + per-layer `NODE_WEIGHTS`. Gens 2–5 use `GEN2_NODE_WEIGHTS` and rival nodes on maps 1/3/5/7.
- `js/battle.js` — the only engine. Round-stepper `runBattleRound` / `executeTurn` via injected `io`. `runBattle` = Tower auto driver; `runInteractiveBattle` (game.js) = campaign. Auto-path rng/event order is load-bearing for Tower replays — don’t reshuffle without a seeded `detailedLog` diff. Struggle is Tower-only (`io.allowStruggle`); campaign mutual-immunity is the deadlock path in `runBattleScreen`.
- `js/endless.js` — Battle Tower. Own traits/tiers and `seededRng`. State is `poke_endless_state`, separate from the campaign run.
- `js/ui.js` — DOM. `showScreen(id)` toggles `<div class="screen">`s in `index.html` (no router). Canvas attack anims on `#battle-anim-canvas`.
- `js/game.js` — mutable `state`, main loop, `onNodeClick` → `doBattleNode` / `doCatchNode` / `doBossNode` / …. Persistence, achievements.
- `js/rules.js` — rules modal **and** `window.POKELIKE_RULES` (mirrored into `#pokelike-llm-spec`). Update when gameplay rules change.
- `js/cloud-save.js` — optional `save.pokelike.xyz`. Game must work when unreachable. Every fetch needs a timeout (`SAVE_FETCH_TIMEOUT_MS`); `initGame()` **awaits** `initCloudSave()`, so a hung fetch freezes the title screen.

Trainer/item sprites live under `sprites/`. Pokémon instances prefer `pokedex.json` `spriteUrl` (PokeAPI CDN) with `sprites/pokemon/{id}.png` as fallback.

## Gens / modes

Normal / Nuzlocke / Battle Tower.

`state.runGen`: `'1'..'5'|'all'`. `getRunGen()` falls back to legacy `gen2Mode` / `bothGens` for old saves. Title UI uses `data-gen="both"` → `runGen: 'all'` (Tot). Unlock: Gen 4 after beating Gen 3, Gen 5 after Gen 4, Tot after beating 1–5 (`isGenBeaten()` reads Hall of Fame).

Per-gen tables live in `GEN_RUN_CONFIG` (data.js). Branch new gen logic there — never new booleans. Keep `gen2Mode` / `bothGens` in sync on `state` for old saves.

## Conventions

- **RNG**: `rng()` (mulberry32 in game.js) for game logic, never `Math.random()`. Seed is saved/restored with the run. `endless.js` has its own `seededRng` for region rolls.
- **Reset safety**: `runGeneration` (game.js) increments on every run (re)start. Capture it at the start of async work (battle loops, anim callbacks) and bail if it changed.
- **Persistence**: `localStorage` keys `poke_*` (`poke_current_run`, `poke_meta`, `poke_dex`, `poke_endless_state`, …). `saveRun()` writes `{...state, currentNodeId, rngSeed}` (drops `currentNode`). Changing `state` shape breaks existing saves. Cross-tab: `runId` + `saveSeq`; a stale tab sets `_tabStale` and stops writing.
