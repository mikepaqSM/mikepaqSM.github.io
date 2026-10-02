# mikepaqSM.github.io

GitHub Pages site (served from `main` at https://mikepaqsm.github.io). It holds app pages
(`cardamom/`, `quikcorder-*.html`; leave these alone unless asked) and the game **Watson Dog**
in `game/`.

## Working with the owner
- The owner is not a programmer and often works from a phone. Explain things in plain language,
  keep replies short, and skip jargon.
- Workflow: commit on the designated `claude/...` branch, push, open a PR to `main`. The owner
  merges on their phone; GitHub Pages deploys in about a minute. Before new work, check whether the
  last PR merged. If it did, restart the branch from `origin/main`; if not, add to the open PR.
- Never commit screenshots or test files. Keep them in the scratchpad.
- Be mindful of usage: batch related changes, and skip heavy testing for trivial tweaks.

## Watson Dog (`game/`)
- `index.html` is the whole game (HTML/CSS/JS in one file, canvas 2D, no build step, no libraries).
- `manifest.webmanifest` and `icon-180/192/512.png` let it be added to the Home Screen and run
  full screen. Icons were rendered from the game's own `drawDog()` art; regenerate them if Watson's
  look changes. The manifest and icon links in `index.html` use absolute `/game/` paths with a
  `?v=N` cache-buster; bump it when they change.

### Story and characters
- **Watson**: orange dog (belly the same orange). The player. Nicknames: **Lovebug**, **Lovie**.
- **Jackie**: Watson's owner, a woman with chin-length blond hair, teal top. Stands by the doghouse.
  Warm and affectionate; uses the nicknames ("Good work, Lovebug!").
- **Fred**: small orange tabby who sells gear from a crate next to the doghouse. Aloof and
  nonchalant, doesn't care about anything, always cold (red scarf, shivers), wants warmth, perks
  up only for the warm Inferno Bone.
- Pollution has turned the town's friendly plant folk mean (enemies: Sproutling, Thornbush,
  Toxic Toadstool, Smog Oak). **No dog-on-dog or dog-on-human violence**: the old human bosses were
  removed (their `LOOKS` art is still there, unused). Only plants are ever attacked.
- **Dog friends** (`FRIENDS`): lost dogs tangled in vines around town; Watson frees them (FREE
  button), brings one along (swap at the doghouse). Ability friends: Splash (swim), Scout (plants back
  off while peeing), Zip (faster, half-cost rolls). Attack friends (own button): Boomer, Frost, Ember. Bond
  (1-3 stars) grows with plants knocked out together. Some friends sit on islands that need Splash.
- **World**: a 120x120 sea with islands (`ISLANDS`, wobbly coastlines via `coastR`). Watson starts on
  Home Island; he hates water (shakes his head, won't go in) until he frees Splash there, then can swim
  to Suburb, Downtown and Pound Isles (plus small islets). Plant toughness comes from `tierAt`.
  A corner minimap shows islands, turf, Watson, home and lost dogs.
- **Turf**: hold the PEE button (or P) to pee; letting go loses an unfinished trail; a trail outside Watson's turf that loops back to the turf (or
  closes on itself) claims everything inside (flood fill on a `G`x-per-tile grid). Getting hit erases
  the trail (so does swimming). The pee meter refills **only at the doghouse**. Plants inside new
  turf are purified; fewer plants spawn as turf grows.
- **Open world, no winning.** Progress = exploring, freeing friends, finding rare items.
- **Rare finds** (`TREASURES`, one per big island, glowing beam) unlock **doghouse upgrades**
  (`UPGRADES`, built from the doghouse menu with XP): Big Water Bowl, Fred's Shop Stall, Comfy Bed,
  Cozy Fireplace, Bigger Doghouse, Fred's Shop, Doghouse Tower (`after` = needs another upgrade
  first). Some finds sit under `OBSTACLES`: boulders (smashed only by Boomer's Ground Pound) and
  brambles (burned only by Ember's Fire Fetch). Fred's stock starts tiny; items tagged
  `shop: 'stall' | 'fire' | 'shop'` appear once those are built. The doghouse and Fred's stand
  visibly change with upgrades. Owner wants to grow this slowly.
- **Outposts** (`OUTPOSTS`): dry water bowls on the other islands; once inside Watson's turf they
  refill his pee like the doghouse does.
- Running costs no stamina; rolling costs stamina and makes Watson untouchable for the whole roll.
- **Outfits** (`OUTFITS`): Fred sells them, wear them at the doghouse. Slots neck/head/body/feet;
  some are armor/stat items, some just style. Drawn in `drawDog` via `o.wear`.
- **Save game**: 3 slots in `localStorage` (`slotKey(n)`, active slot in `watsonDogActive`). Autosaves the
  active slot every 10s and when the app is hidden; loads on start with Watson at the doghouse. The
  pause button (or Esc) opens a pause menu: Resume, Save, Save here / Load / New per slot (switching
  reloads the page into that slot, skipping the title). "New game" on the title wipes the active slot.
  When adding new progress state, add it to `saveGame`/`loadGame`.
- **Resting**: Watson licks Jackie, then rolls over for belly rubs (`startRest`/`updateScene`,
  `drawDogBellyUp`, Jackie `kneel`/`bend`), then the doghouse menu opens.

### Design rules the owner cares about
- Souls-like but fair: telegraphed attacks (red circle), dodge roll, stamina, punish window
  (yellow ring = 2x critical), XP dropped on knock-out and recoverable. **Not button-mashy**:
  enemies die in few hits, and Watson can't take many either. Difficulty has been eased twice,
  so lean forgiving. XP was slowed down once (plants give half, levels and upgrades cost more).
- **Minimal on-screen text.** No instruction screens; players learn through one-time hints
  (`HINTS` list). The HUD is compact. Jackie and Fred speak in speech bubbles via `speak()`.
- Start screen is just the Watson Dog logo.
- Phone first. Landscape, joystick on the left half, buttons on the right. Only the buttons
  attack or interact (wooden/brass round buttons with SVG icons; a dark wedge sweeps while a move
  recharges; small ability buttons only appear once earned and fill `ABILITY_SLOTS` in order); the big action button shows REST / TRADE near the doghouse / Fred, otherwise
  the gear's verb (BITE, WHACK, BONK...). No zooming. Desktop: WASD, click attacks toward the
  mouse, Space rolls, Shift runs.
- **Audio: the owner will make all SFX and music themselves. Never generate or add placeholder
  sounds.** When files arrive (planned: `game/audio/`, MP3), wire them in: unlock on first tap,
  mute toggle, crossfade music by zone or boss.

### Code map (`game/index.html`)
Sections are marked with `// ---------- Name ----------` comments:
- **Game data**: `WEAPONS` (fetch gear with `verb`), `SKILLS`, `STATS`, `ENEMY_TYPES` (plants),
  `FRIENDS`, `ISLANDS`, `OUTFITS`. All balance numbers live here.
- **State**: `player`, plus positions of `JACKIE`, `FRED`, and the doghouse at the map `CENTER`.
- **Overlays**: title, shop (`renderShop` for the doghouse level-up, `renderFredWares` for Fred),
  death, win. Dialogue line lists (`FRED_*`, `JACKIE_*`).
- **Input**: keyboard, mouse, touch joystick (`joy`), phone buttons (`touchButton`,
  `updateTouchButtons`), full screen, and no-zoom handlers.
- **Combat / Update**: enemy AI states (idle, chase, windup, recover, return), boss combos and
  leash, relic effects.
- **Ground**: the sea is a flat fill; land is drawn in CHxCH-tile chunks cached in small canvases
  (`renderChunk`, at most 24 kept), redrawn via `dirtyChunks()` when turf changes. Off-screen
  enemies and friends aren't drawn.
- **Character art**: everything is drawn with canvas shapes. `drawDog` (breeds via `look`), `drawPerson` (+ `LOOKS`),
  `drawPlant`, `drawFred`, `drawDoghouse`. `paintMode` handles hit flash and wind-up tint.
- **Learn-as-you-play hints**, the logo renderer, and the main loop are at the end.

### Testing
Chromium is pre-installed for Playwright (`/opt/node22/lib/node_modules/playwright`,
`PLAYWRIGHT_BROWSERS_PATH=/opt/pw-browsers`). Load `file:///.../game/index.html`. Press a key to
leave the title screen, drive game state via `page.evaluate`, and check desktop plus iPhone
emulation (`devices['iPhone 13']`, landscape 844x390 and portrait). Check for `pageerror`s and
look at screenshots for visual changes.
