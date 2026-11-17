# Lumen

A minimal, mobile-first planet-capture swarm game in the spirit of Auralux. You start with one planet and a handful of ships; take every planet on the map to win.

The whole game is one self-contained file, `index.html` (HTML, CSS and JavaScript, no build step, no dependencies beyond a Google Font).

## Run it

Open `index.html` in any modern browser, or serve the folder:

```
python3 -m http.server 8000
```

then visit `http://localhost:8000`. It works with touch and with a mouse.

## How to play

| Action | Touch | Mouse |
|---|---|---|
| Flag a grey or rival planet for attack | Tap it | Click it |
| Commit more to a flag (up to 3 dots) | Tap it again | Click again |
| Cancel a flag | Hold the planet | Hold, or right-click |
| Select one planet's ships | Tap your planet | Click your planet |
| Send the selected ships | Tap a planet or a spot | Click |
| Mark your planet for defence | Hold your planet | Hold |
| Upgrade a planet | Select it, press Upgrade | Same |
| Move around the map | Drag | Drag |
| Zoom | Pinch, or the ⤢ button | Scroll wheel |
| Pause | II button | Space or Esc |

### Flags

You command targets, not individual ships. A flag pulls ships from your nearest planets and keeps reinforcing until the target is yours.

| Dots | Planets that join | Ships each keeps home |
|---|---|---|
| 1 | 2 nearest | 12 |
| 2 | 3 nearest | 5 |
| 3 | 4 nearest | 0 |

A planet with more than three rival ships nearby keeps its ships rather than sending them. Cancelling a flag recalls the ships it sent.

### Automatic behaviour

- **Auto supply:** planets far from any grey or rival planet pass all but 6 ships to the next planet closer to the front.
- **Strays:** ships parked in open space drift to your nearest planet after about six seconds.
- **Opportunistic fire:** any ship within range of a grey or rival planet shoots it, whatever its orders.

## Rules and numbers

All values are constants near the top of the script in `index.html`.

| Thing | Value | Constant |
|---|---|---|
| Planet health when owned | 130 | `PLANET.hp` |
| Shots to capture a grey planet | 100 | `PLANET.cost` |
| Base build time per ship | 1.5 s | `PLANET.rate` |
| Upgrade costs (level 2, level 3) | 20, 35 ships | `UPC` |
| Build speed per upgrade level | +50% of base | `spawnRate()` |
| Ship health | 4 hits | `UHP` |
| Ship fire interval (vs ships / vs planets) | 0.7 s / 2 s | `FIRE`, `PFIRE` |
| Ship speed | 58 units/s | `SPEED` |
| Fleet cap per side | 400 | `CAP` |
| Garrison limit per planet (level 1 / 2 / 3) | 40 / 60 / 80 | `GCAP` |
| Starting ships | 20 | `START` |
| Global time scale | 0.8 | `TS` |

- A planet stops building while its garrison is at its limit; the small count under its health shows garrison and limit. Auto supply stops feeding a planet once it holds 1.5 times its limit.
- A captured planet starts at half health and level 1. Owned planets regenerate after 4 seconds without being hit.
- A rival planet shot to zero turns grey and must then be shot down again to capture.
- Grey planets never shoot back, and neither do owned planets; only ships defend.
- A side is eliminated when it holds no planets. You lose when you hold no planets and have no ships.

## Modes and difficulty

- **Campaign:** eight maps, unlocked in order, with an Easy / Normal / Hard setting. Maps after the first add a few randomly placed grey planets each time.
- **Skirmish:** random map, 1 to 3 rivals, three map sizes, four rival levels.

| Rival level | Decision interval per planet | Build speed vs you | Upgrades |
|---|---|---|---|
| Gentle | ~9 s | 60% | No |
| Calm | ~6 s | 85% | Yes |
| Even | ~4 s | 100% | Yes |
| Fierce | ~2.5 s | 100% | Yes |

Each rival planet decides for itself at its own moment: it picks a nearby target by weighted chance and sends a random share of its spare ships, sometimes setting a standing route for its new ships (`aiPlanet()`).

## Stats

The HUD shows live ship and planet counts for every side. The pause and end screens show, per side: ships now, largest fleet, ships built, built per minute, ships lost, ships destroyed, destroyed per loss, shots fired, planet damage dealt, planets now, most planets held, planets captured, planets lost and upgrades bought. The end screen also charts fleet size over time.

## Records

Every campaign map or skirmish played to the end is added to a career record, opened from **View your records** on the menu: totals (games, wins, streaks, fastest win, ships built, lost and destroyed, planets captured and lost), how you play (flags, manual sends, defend flags, upgrades and captures per game), win rate by rival level, and the last 30 games. Records are stored in the browser's `localStorage` under `lumen.career`, so they stay on the device and browser you played on.

## Code map

Everything lives in one script inside `index.html`, in this order:

| Section | What it holds |
|---|---|
| constants | Tuning values, difficulty table, campaign map data (`LEVELS`) |
| view | Canvas sizing and the camera (`cam`, `applyCam`, `zoomAt`) |
| game state | `newGame`, planets, ships, `order`, `hitPlanet`, orbit ring layout |
| simulation | `step`: building, targeting, movement, combat, planet avoidance |
| flags, supply, strays | `flagTick`, `cancelFlag`, `nearestOwn` |
| rivals | `aiPlanet` |
| drawing | `draw`, `tri` |
| HUD | Bars, chips, `statsTable`, `drawChart`, upgrade button |
| input | Pointer handling: tap, hold, pan, pinch, wheel |
| screens | Menu, pause, end screen, campaign progress |

Campaign maps are lists of `[u, v, size, owner]` with `u` along the long axis of the screen and `v` along the short axis (both 0 to 1), so the same map fits portrait and landscape. Owner `0` is the player, `1`–`3` are rivals, `-1` is grey.

Campaign progress and menu settings are saved in the browser's `localStorage`.

## Status

Game logic is checked by running headless simulated matches; feel and balance are tuned by hand and still changing.

## Ideas not built yet

- Planets that shoot back at attackers
- Upgrade choices (faster building or tougher health)
- Special planets, ship types, terrain, fog of war
- Campaign goals beyond "take everything"
- Sound and haptics
- Two players
