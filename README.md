# Pokelike (fork)

A Pokémon roguelike, forked from https://pokelike.xyz/ to play and modify locally.

**▶ Play the demo: https://pcasaspere.github.io/pokelike_v2/**

## How to play

You need a static server (it won't open via `file://` due to browser restrictions):

```bash
python3 -m http.server 8000
```

Then open http://127.0.0.1:8000/ in your browser.

## Structure

- `index.html` — entry point, all the screen markup
- `css/style.css` — styles
- `js/` — game logic:
  - `data.js` — Pokémon, moves and item data
  - `map.js` — map / node generation
  - `battle.js` — battle system
  - `endless.js` — Battle Tower mode
  - `ui.js` — interface and screens
  - `game.js` — main loop and state
  - `rules.js` — human-readable rules + machine-readable LLM spec
  - `cloud-save.js` — cloud save (optional, points to save.pokelike.xyz)
- `ui/`, `sprites/` — local images (backgrounds, buttons, etc.)

## Notes

- Pokémon and trainer sprites are loaded at runtime from external CDNs (PokeAPI and
  Pokémon Showdown), so an internet connection is required.
- Game saves use the browser's `localStorage`.
- `cloud-save.js` talks to an external server; the game works fine without it.

## Modifying

Edit any file under `js/` or `css/` and reload the page (Cmd+Shift+R to bypass the
cache). For example, battle values in `js/battle.js` or Pokémon data in `js/data.js`.

## Disclaimer

Unofficial fan-made project. Not affiliated with, endorsed by, or
sponsored by Nintendo, Game Freak, or The Pokémon Company.
Pokémon and related names and characters are trademarks of their
respective owners.

Fork of [pokelike.xyz](https://pokelike.xyz/).
