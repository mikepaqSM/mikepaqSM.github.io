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
- **Watson**: all-orange dog (no brown spots or saddle; belly the same orange) with a tiny white chest patch. The player. Nicknames: **Lovebug**, **Lovie**.
- **Jackie**: Watson's owner, a woman with chin-length blond hair, teal top. Stands by the doghouse.
  Warm and affectionate; uses the nicknames ("Good work, Lovebug!").
- **Fred**: small orange tabby who sells gear from a crate next to the doghouse. Aloof and
  nonchalant, doesn't care about anything, always cold (red scarf, shivers), wants warmth, perks
  up only for the warm Inferno Bone.
- Pollution has turned the town's friendly plant folk mean (enemies: Sproutling, Thornbush,
  Toxic Toadstool, Smog Oak; plus giant plant bosses). **No dog-on-dog or dog-on-human violence**: the old human bosses were
  removed (their `LOOKS` art is still there, unused). Only plants are ever attacked.
- **Dog friends** (`FRIENDS`): lost dogs tangled in vines around town; Watson frees them (FREE
  button), brings one along (swap at the doghouse). Ability friends: Splash (swim rivers/lakes), Scout (plants back
  off while peeing), Zip (faster, half-cost rolls). Attack friends (own button): Boomer, Frost, Ember. Bond
  (1-3 stars) grows with plants knocked out together. Some friends sit behind gates or water.
- **World**: one big continent (`MAP` 180, sea around it, `CONTINENT`) split into `REGIONS`: Home Meadow
  (start), The Suburbs, Lakeside, Downtown, The Pound District. Region borders (`BORDERS`) are cliffs by
  default (impassable rock walls), except: home-suburbs open, home-lakeside a river (needs Splash to swim),
  home-downtown a boulder gate (Boomer's headbutt), suburbs-pound a bramble gate (Ember's Fire Breath).
  `GATES` become big `OBSTACLES`. `LAKES` add swim spots (one hides a treasure on an islet). Terrain is a
  grid (`terrainAt`: `T_LAND`/`T_WATER`/`T_CLIFF`/`T_SEA`); `moveBody` blocks cliffs/sea always and water
  without Splash (Watson shakes his head). `ISLANDS` / `islandAt` are aliases for regions. Plant toughness
  comes from `tierAt`. A corner minimap shows land, cliffs, water, turf, Watson, home and lost dogs.
  Progression: Splash (home) -> Lakeside; Boomer (suburbs) -> Downtown; Ember (downtown) -> Pound.
- **Trees** (`TREES`, seeded groves + loners, density per region in `DENS`, pines `PINE`, bare trees in the
  Pound): trunks block movement (`treeAt`, bucketed in `treeGrid`; checked in `moveBody` and `okSpot`),
  kept clear of friends, finds, nests, gates and boss arenas. `drawTree`: smoggy colours outside turf,
  lush with blossoms inside (bare ones leaf out); fades when Watson is behind it.
- **Turf**: hold the PEE button (or P) to pee; letting go loses an unfinished trail; a trail outside Watson's turf that loops back to the turf (or
  closes on itself) claims everything inside (flood fill on a `G`x-per-tile grid). Getting hit erases
  the trail (so does swimming). **Pee never refills until Watson trades the Shiny Bowl to Fred**
  (`built.has('bowl')`, `peeRefills()`); after that, resting refills it (and claimed outpost bowls). Plants inside new
  turf are purified; fewer plants spawn as turf grows. Claimed land sprouts flowers (`flowerAt`,
  `drawFlower`): new ones pop up over ~1.4s (`growing`, `drawBlooms`), then get baked into the chunks.
- **Open world, no winning.** Progress = exploring, freeing friends, finding rare items.
- **Doghouse upgrades** (`UPGRADES`) are tied to territory: **Home XP** (`homeXp`, 1 per tile of land
  claimed with pee, a running total that's never spent; little house in the HUD). Each upgrade unlocks
  at a Home XP threshold (`home`), some also need a rare find (`needs`, from `TREASURES`, glowing beam),
  some need another upgrade first (`after`); then it's built free from the doghouse menu: Flower Garden,
  Fred's Shop Stall, Bigger Doghouse, Comfy Bed, Fred's Shop, Cozy Fireplace, Doghouse Tower. Regular XP
  only fills the level bar. Some finds sit under `OBSTACLES`: boulders (smashed only by Boomer's headbutt) and
  brambles (burned only by Ember's Fire Breath). Fred's stock starts tiny; items tagged
  `shop: 'stall' | 'fire' | 'shop'` appear once those are built. The doghouse and Fred's stand
  visibly change with upgrades. Owner wants to grow this slowly.
- **Smog nests** (`NESTS`, 3-4 per region, seeded positions): purple bubbling mounds.
  A nest inside turf turns into a flowering bush. When every nest in a region is in turf the region is
  clean (`islandClean`): no new plants spawn there (`okSpot` checks `cleanAt`), but the plants already
  there are NOT removed: each must be beaten. They come back after a knock-out (`stragglers`, saved)
  until beaten. Minimap
  shows nests (purple / green).
- **Outposts** (`OUTPOSTS`): dry water bowls in the other regions; once inside Watson's turf (and after
  the Fred bowl trade) they refill his pee.
- Attacks are snappy (strike `cd` 0.4, `lock` 0.18, `swingMax` 0.2); owner asked for less delay.
- **Special moves**: Zoomies (`whirl`) = Watson runs one quick loop (`player.zoom`, `ZOOM_T`), untouchable,
  hitting each plant he passes for 1.5x, with a wind swirl (`drawWind`). Boomer's move is a short charge +
  headbutt (`jab`, max 2 tiles, small hop; owner said the old leap flew too far). Ember breathes fire
  (`breath` act spawns `flame` projectiles drawn by `drawFlame`; each plant burned once per breath).
- Running costs no stamina; rolling costs stamina and makes Watson untouchable for the whole roll.
- **Leveling**: XP fills a level bar (`gainXp`), but Watson **only levels up when he rests** at the
  doghouse (`levelUpAtRest`, end of the rest scene); unbanked XP is what drops on knock-out. Each level
  gives one pick (`freePicks()` = level - 1 - stat points) spent at the doghouse Level up tab on a stat.
- **Fred trades only for shiny trinkets**, which are very rare: a handful hidden on the map
  (`TRINKETS`, 1-2 per region, seeded positions, `gotTrinket` ids) plus rare plant drops
  (`DROP_CHANCE` by tier, 2-10%; `drops`, saved). Prices (`tk`) are 2-3 for small things, **5 for
  powerful items**. Never XP. HUD shows a blue gem with the count. Bosses drop piles of them.
- **Bosses** (`BOSSES`): giant polluted plants, one per region (Bramble King, Mother Toadstool,
  Smog Titan, Wilted Giant, Bog Monarch), drawn with `drawPlant` at large scale plus a purple glow. They chain
  attacks, don't stagger, stay within 8 tiles of home, aren't purified by turf, and stay beaten
  (`bossesBeaten`, saved). Each drops `loot` trinkets in a ring.
- **Outfits** (`OUTFITS`): Fred sells them, wear them at the doghouse. Slots neck/head/body/feet;
  some are armor/stat items, some just style. Drawn in `drawDog` via `o.wear`.
- **Save game**: 3 slots in `localStorage` (`slotKey(n)`, active slot in `watsonDogActive`). Save version `v: 2` (the land rework dropped v1 saves). Autosaves the
  active slot every 10s and when the app is hidden; loads on start with Watson at the doghouse. The
  pause button (or Esc) opens a pause menu: Resume, Save, Save here / Load / New per slot (switching
  reloads the page into that slot, skipping the title). "New game" on the title wipes the active slot.
  When adding new progress state, add it to `saveGame`/`loadGame`.
- **Resting**: Watson licks Jackie, then rolls over for belly rubs (`startRest`/`updateScene`,
  `drawDogBellyUp`, Jackie `kneel`/`bend`), then the doghouse menu opens.

### Design rules the owner cares about
- Souls-like but fair: telegraphed attacks (red circle), dodge roll, stamina, punish window
  (yellow ring = 2x critical), XP dropped on knock-out and recoverable. **Not button-mashy**:
  enemies die in few hits, and Watson can't take many either. Difficulty was eased twice, then the
  owner found it too easy after the land rework, so plants/bosses got tougher and progression slower
  (levelCost 60*1.27^n, upgrade Home XP thresholds up ~60%, lower trinket drops). Tune in small steps.
- **Minimal on-screen text.** No instruction screens; players learn through one-time hints
  (`HINTS` list). The HUD is compact. Ready-to-spend is shown by pulsing HUD badges, not hint text:
  a gold up-arrow by "Lv" (`canLevelUp`), a glow on the house icon (`canBuildUpgrade`), and a pulsing
  gold home dot on the minimap. Jackie and Fred speak in speech bubbles via `speak()`.
- Start screen is just the Watson Dog logo.
- Phone first. Landscape, joystick on the left half, buttons on the right. Only the buttons
  attack or interact (wooden/brass round buttons with SVG icons; a dark wedge sweeps while a move
  recharges; small ability buttons only appear once earned and fill `ABILITY_SLOTS` in order); the big action button shows REST / TRADE near the doghouse / Fred, otherwise
  the gear's verb (BITE, WHACK, BONK...). No zooming. Desktop: WASD, click attacks toward the
  mouse, Space rolls, Shift runs.
- **Audio: the owner makes all SFX and music themselves. Never generate or add placeholder
  sounds.** The **Audio** section (`SOUNDS`, `MUSIC`) loads `game/Audio/Watson Dog_<name>.mp3` (capital A; names like `SFX_Bark_01`, `MX_Theme_01`) with Web Audio,
  unlocking on the first tap. Each sound has several takes played round robin (`sfx(kind)`): attack
  (Watson's hit lands), death (plant knocked out), bark (Watson gets hit), fred (walking up to Fred),
  click (menu buttons, 2 takes), swipe (attack that misses), special (freeing a friend or finding a rare
  item). `music` loops. Missing files are skipped silently. Mute toggle in the
  pause menu (`watsonMute`). Later: more music, crossfade by region.

### Code map (`game/index.html`)
Sections are marked with `// ---------- Name ----------` comments:
- **Game data**: `WEAPONS` (fetch gear with `verb`), `SKILLS`, `STATS`, `ENEMY_TYPES` (plants),
  `FRIENDS`, `ISLANDS`, `OUTFITS`. All balance numbers live here.
- **State**: `player`, plus positions of `JACKIE`, `FRED`, and the doghouse at the map `CENTER`.
- **Overlays**: title, shop (`renderShop`: tabs per mode, one short line per item; doghouse = Level up /
  Doghouse / Friends / Wardrobe, Fred = Gear / Tricks / Treats / Outfits; owner wants menus low on text),
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
