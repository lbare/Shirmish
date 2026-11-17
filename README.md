# Lumen

A minimal, mobile-first planet-capture swarm game in the spirit of Auralux. You start with one planet and a handful of ships; take every planet on the map to win.

The whole game is one self-contained file, `index.html` (HTML, CSS and JavaScript, no build step, no dependencies beyond a Google Font).

## Run it

Open `index.html` in any modern browser, or serve the folder:

```
python3 -m http.server 8000
```

then visit `http://localhost:8000`. It works with touch and with a mouse.

### Install on a phone

The repo includes a web app manifest and icons. Host the folder over HTTPS (for example GitHub Pages: Settings → Pages → deploy from `main`), open it on the phone, then use **Add to Home Screen** (iOS Safari share menu) or **Install app** (Android Chrome). It opens full screen like an app. There is no offline cache.

## How to play

The whole map always fits the screen.

| Action | Touch | Mouse |
|---|---|---|
| Flag a grey or rival planet for attack | Tap it | Click it |
| Commit more to a flag (up to 3 dots) | Tap it again | Click again |
| Cancel a flag | Tap the × on the planet | Click the ×, or right-click |
| Send one planet's ships now | Drag from your planet to a target | Same |
| Select one planet's ships | Tap your planet, then tap a target | Same |
| Send every ship you have | Double-tap a planet | Double-click |
| Mark your planet for defence | Hold your planet | Hold |
| Upgrade a planet | Select it, press Upgrade | Same |
| Pause | II button | Space or Esc |

### Flags

A flag pulls ships from your nearest planets and keeps reinforcing until the target is yours. Ships leave in a few waves rather than all at once.

| Dots | Planets that join | Ships each keeps home |
|---|---|---|
| 1 | 2 nearest | 12 |
| 2 | 3 nearest | 5 |
| 3 | 4 nearest | 0 |

A planet with more than three rival ships nearby keeps its ships rather than sending them. Cancelling a flag recalls the ships it sent.

### Automatic behaviour

- **Auto supply:** planets far from any grey or rival planet pass all but 6 ships to the next planet closer to the front, until that planet holds 1.5 times its garrison limit.
- **Strays:** ships parked in open space drift to your nearest planet after about six seconds.
- Ships only shoot a planet they were sent to; ships passing by leave it alone.

## Rules and numbers

All values are constants near the top of the script in `index.html`.

| Thing | Standard planet | Large planet | Constant |
|---|---|---|---|
| Health when owned | 130 | 210 | `PLANET.hp`, `BIG.hp` |
| Shots to capture when grey | 100 | 170 | `.cost` |
| Base build time per ship | 1.5 s | 1.05 s | `.rate` |
| Garrison limit (level 1 / 2 / 3) | 40 / 60 / 80 | 60 / 90 / 120 | `GCAP`, `gcap()` |

| Thing | Value | Constant |
|---|---|---|
| Upgrade costs (level 2, level 3) | 20, 35 ships | `UPC` |
| Per upgrade level | +50% build speed, +35% health, bigger garrison, faster turret | `spawnRate()`, `maxHp()`, `TURRET` |
| Planet turret interval (level 1 / 2 / 3) | 0.6 / 0.45 / 0.35 s | `TURRET` |
| Ship health | 6 hits | `UHP` |
| Ship fire interval (vs ships / vs planets) | 0.7 s / 2 s | `FIRE`, `PFIRE` |
| Ship speed | 58 units/s (halved in asteroid belts) | `SPEED` |
| Fleet cap per side | 400 | `CAP` |
| Starting ships | 20 | `START` |
| Global time scale | 0.8 | `TS` |

- A captured planet starts at half health and level 1. Owned planets regenerate after 4 seconds without being hit, and shoot rival ships in range.
- A rival planet shot to zero turns grey and must then be shot down again to capture.
- Grey planets never shoot back, but recover 2 points a second after 5 seconds without being hit.
- A planet stops building while its garrison is full. Groups of 30 or more ships at a planet are drawn as larger ships standing for 5 (or 10, from 150 up); this is visual only.
- **Asteroid belts** halve ship speed. **Nebulae** hide the ships inside from the other side unless one of its own ships is right next to them.
- **Fog of war** (skirmish option): rival ships and planet details are only visible near your planets and ships.
- A side is eliminated when it holds no planets. You lose when you hold no planets and have no ships.

## Modes and difficulty

- **Campaign:** eight fixed maps, unlocked in order, with an Easy / Normal / Hard setting. They are lopsided on purpose: rivals start with 4 more ships per map number, and from map 5 their home planets start upgraded.
- **Skirmish:** generated map, 1 to 3 rivals, three sizes, four rival levels, terrain and fog switches.
- **Map codes:** every skirmish map has a number, shown on the pause and end screens. Enter it under Map code to replay that layout. **Today's map** uses the date as the code. The same code gives the same map only on a screen of similar shape, because maps are fitted to the screen.

| Rival level | Decision interval per planet | Build speed vs you | Upgrades |
|---|---|---|---|
| Gentle | ~8 s | 70% | No |
| Calm | ~5 s | 95% | Yes |
| Even | ~3.2 s | 110% | Yes |
| Fierce | ~2 s | 130% | Yes |

Each rival planet decides for itself at its own moment (`aiPlanet()`), and sometimes just waits. Each rival also has a style: **Aggressive** (keeps few ships home, prefers attacking), **Defensive** (keeps more home, upgrades more) or **Expansionist** (prefers grey planets). With two or more rivals, they prefer attacking each other over you.

## Stats

The HUD shows live ship and planet counts for every side (hidden for rivals under fog). The pause and end screens show, per side: style, ships now, largest fleet, ships built, built per minute, ships lost, ships destroyed, destroyed per loss, shots fired, planet damage dealt, planets now, most planets held, planets captured, planets lost and upgrades bought. The end screen also charts fleet size over time.

## Records and achievements

Every campaign map or skirmish is added to a career record, opened from **View your records** on the menu: 13 achievements, totals, how you play (flags, direct sends, defend flags, upgrades and captures per game), win rate by rival level, and the last 30 games with their map codes. A game you quit or restart after the first 15 seconds counts as a loss.

Records are stored in the browser's `localStorage` (`lumen.career`). When the page runs as a Claude artifact with the `db` and `user` capabilities declared, the same record is also saved per signed-in person and follows them between devices (`cloudInit()`); elsewhere that code does nothing.

You can pick your colour on the menu; rivals take the remaining colours.

## Code map

Everything lives in one script inside `index.html`, in this order:

| Section | What it holds |
|---|---|
| constants | Tuning values, difficulty table, campaign map data (`LEVELS`) |
| storage | Progress, career record, optional cloud sync |
| view | Canvas sizing; the map is always fitted to the screen |
| game state | `newGame` (seeded map, terrain, head starts), planets, ships, `order`, `sendStag`, `hitPlanet`, orbit ring layout |
| simulation | `step`: building, turrets, terrain, visibility, targeting, movement, combat, planet avoidance |
| flags, supply, strays | `flagTick`, `cancelFlag`, `nearestOwn` |
| rivals | `aiPlanet` |
| drawing | `draw`, `tri` |
| HUD | Bars, chips, `statsTable`, `drawChart`, upgrade button |
| input | Pointer handling: tap, double-tap, hold, drag-to-send |
| screens | Menu, pause, end screen, records, achievements (`ACH`), campaign progress |

Campaign maps are lists of `[u, v, size, owner]` with `u` along the long axis of the screen and `v` along the short axis (both 0 to 1), so the same map fits portrait and landscape. Owner `0` is the player, `1`–`3` are rivals, `-1` is grey.

Campaign progress and menu settings are saved in the browser's `localStorage`.

## Status

Game logic is checked by running headless simulated matches; feel and balance are tuned by hand and still changing.

## Ideas not built yet

- Upgrade choices, special planets, more ship types
- Campaign goals beyond "take everything"
- Sound
- Two players
