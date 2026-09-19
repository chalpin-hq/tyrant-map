# Tyrant Map Planner

Interactive hex map planning tool for the Tyrant mobile game. Design the terrain once, then let your guild place and drag Hub / Outpost / City markers with territory overlays, sharing maps via JSON export/import.

## Two modes

- **Design mode** (`index.html?design`) — for you, the map author. Unlocks terrain painting (the categorized swatch palette + "Terrain" tool) and a full "Clear" that wipes terrain too. Use this to lay down the background art once.
- **Final/play mode** (`index.html`, no query string) — what you hand to guildmates. Terrain is locked and read-only; only Hub, Outpost, and City can be placed, dragged to reposition, or erased. "Clear" here only removes markers, never terrain.

Both modes default to a **"Pan" tool** — nothing is placed, painted, or erased until you deliberately click Hub/Outpost/City/Terrain/Erase, so panning and zooming around are always safe. Press **Esc** any time to deselect back to Pan.

## The default map lives in the repo

`map-data/default.json` is the committed baseline map — `index.html` automatically loads it at boot, in either mode. Terrain painted in a live session only exists in memory until you save it back to this file:

1. Open `index.html?design` (served over http, see "Running it" below — a plain `file://` double-click can't fetch this file) and paint the terrain, plus place any starting Hub/Outpost/City you want everyone to start with.
2. Click **💾 Save Default Map**. This downloads `default.json`.
3. Move that download over `map-data/default.json` and commit it.

From then on, anyone who opens `index.html` — you, a collaborator, or a guildmate — automatically gets that terrain, no Import step required. Because it's a small, plain JSON file, it's easy to review in a diff, and anyone comfortable with JSON can hand-edit it directly instead of going through design mode.

The plain "↓ Export" / "↑ Import" buttons still exist too, for ad-hoc backups or for a guildmate to share their own Hub/Outpost/City layout with someone else — they just aren't tied to the well-known `map-data/default.json` path, so they don't auto-load.

## Repo structure

```
index.html            ← the whole app (single file, no dependencies)
assets/               ← terrain tile images (optional), one PNG per TERRAIN key —
                        see the full list of expected filenames in "Adding terrain images"
assets/dominions/     ← Dominion icons (optional), one PNG per Dominion label —
                        see "Adding Dominion icons" for the filename convention
map-data/
  default.json        ← the committed baseline map — auto-loaded at boot (see below)
README.md
```

## Running it

You need a local server (or GitHub Pages) for terrain images and `map-data/default.json` to load — browsers block both `fetch()` and `<img>` loads of local files over plain `file://`. The quickest way:

```bash
# Python
python3 -m http.server 8080

# Node
npx serve .
```

Then open `http://localhost:8080`. (A `.claude/launch.json` is included if you're using Claude Code's browser preview — it starts the same Python server for you.)

Opening `index.html` directly by double-clicking it still works for placing/moving Hub/Outpost/City — you'll just see plain terrain colours instead of art, and won't get the committed default map.

## GitHub Pages (share with guild)

1. Push this repo to GitHub
2. Go to **Settings → Pages → Branch: main → / (root)**
3. Your map is live at `https://<you>.github.io/<repo>/`
4. Guildmates just open the URL — `map-data/default.json` and any terrain images load automatically, no Import step. Push an updated `default.json` (see "The default map lives in the repo") whenever the design changes.

## Adding terrain images

`index.html`'s `TERRAIN` object (near the top of the `<script>` block) already has an `img` path wired up for every tile — `assets/<key>.png` — so dropping a correctly-named PNG into `assets/` picks it up with no code changes. Until an image exists for a key, that tile just renders its placeholder `fill` colour (a missing image fails silently to the fallback colour, it won't break the page).

The full tile list, grouped by zone theme, with each expected filename:

**Core (ash / lava wasteland)**
| Key / filename | Label | Blocks placement? |
|---|---|---|
| `deadlands` | Deadlands | no |
| `deadRocks` | Dead Rocks | no |
| `smallLavaPits` | Small Lava Pits | no |
| `lavaPits` | Lava Pits | no |
| `volcano` | Volcano | **yes** |
| `lavaField` | Lava Field | **yes** |
| `lavaPool` | Lava Pool | **yes** |
| `deadForest` | Dead Forest | no |
| `burningForest` | Burning Forest | no |
| `rockPillars` | Rock Pillars | no |
| `deadMountains` | Dead Mountains | **yes** |

**Midlands (green / desert / snow)**
| Key / filename | Label | Blocks placement? |
|---|---|---|
| `mountains` | Mountains | **yes** |
| `forest` | Forest | no |
| `plains` | Plains | no |
| `river` | River | **yes** |
| `desert` | Desert | no |
| `desertOasis` | Desert Oasis | no |
| `desertPebbles` | Desert Pebbles | no |
| `desertMountains` | Desert Mountains | **yes** |
| `snowyPlains` | Snowy Plains | no |
| `snowyForest` | Snowy Forest | no |
| `snowyRocks` | Snowy Rocks | no |
| `snowyMountains` | Snowy Mountains | **yes** |
| `frozenRiver` | Frozen River | **yes** |
| `frozenPond` | Frozen Pond | no |

So e.g. `deadRocks.png` goes in `assets/` and lights up the `deadRocks` tile automatically. To rename a tile, key, or its `block`/`fill`/`stroke` values, edit its entry in `TERRAIN` directly.

**Image tips:**
- Square images work best (64×64 or 128×128 px)
- Tileable textures look great at low zoom
- The image is clipped to the hex shape automatically — art does not bleed into neighboring hexes
- The `fill` colour still shows as fallback if the image fails to load

**Blocking placement:** each terrain type has a `block` flag — `true` means a Hub/Outpost/City can't be placed or dragged onto it (see the tables above for the current defaults). Adjust these per your game's rules right next to the `img` field.

## Adding Dominion icons

Same idea as terrain images: drop a correctly-named PNG into `assets/dominions/` and it lights up automatically, no code change needed. By default the filename is derived from the Dominion's **label** — lowercase it, drop apostrophes, and replace every other run of non-letters/digits with a single hyphen — so two Dominions sharing a label (e.g. the two "Fertile Sanctuary" nodes) share one file, matching how the game itself reuses art per Dominion type.

A few Dominions need to break that rule — either because same-labelled instances actually look different per biome, or because several *different* labels should share one icon. For those, that specific Dominion's entry in `map-data/default.json` carries an explicit `"icon": "<key>"` field that overrides the label-derived filename — set by hand, not through the UI, since it only applies to a handful of entries:

| Filename | Label(s) | Why it's special |
|---|---|---|
| `abandoned-mine-desert.png` | Abandoned Mine (80, 203) | different biome per instance |
| `abandoned-mine-snow.png` | Abandoned Mine (170, 47) | different biome per instance |
| `stonemasons-hall-desert.png` | Stonemason's Hall (170, 203) | different biome per instance |
| `stonemasons-hall-snow.png` | Stonemason's Hall (125, 35) | different biome per instance |
| `watchtower.png` | Eastern/Northwestern/Southwestern Watchtower | one generic icon, three different labels |

Every other Dominion just uses its plain label-derived filename:

| Filename | Label(s) |
|---|---|
| `ancient-forge.png` | Ancient Forge |
| `crumbling-rampart.png` | Crumbling Rampart |
| `crystal-mine.png` | Crystal Mine |
| `drill-yard.png` | Drill Yard |
| `fertile-sanctuary.png` | Fertile Sanctuary *(×2)* |
| `forgotten-library.png` | Forgotten Library |
| `gravel-pits.png` | Gravel Pits |
| `healers-refuge.png` | Healer's Refuge |
| `herbalists-hut.png` | Herbalist's Hut |
| `hermits-study.png` | Hermit's Study |
| `homestead-fields.png` | Homestead Fields |
| `iron-bastion.png` | Iron Bastion |
| `militia-camp.png` | Militia Camp |
| `old-barracks.png` | Old Barracks *(×2)* |
| `prospectors-claim.png` | Prospector's Claim |
| `quarry-camp.png` | Quarry Camp |
| `ruined-archive.png` | Ruined Archive |
| `surveyors-camp.png` | Surveyor's Camp |
| `the-throne.png` | The Throne |
| `timber-mill.png` | Timber Mill |
| `trading-post.png` | Trading Post |
| `verdant-grove.png` | Verdant Grove |
| `war-monument.png` | War Monument |
| `watchtower-ruins.png` | Watchtower Ruins |
| `wayside-shrine.png` | Wayside Shrine |

If you rename an ordinary Dominion or add a new one, the rename/recolour dialog (click a Dominion without dragging it) shows the exact filename it's currently looking for, live, under the Icon row. For a Dominion with an explicit `icon` override, that hint stays fixed on the override's filename regardless of what you type in the label field, since renaming doesn't change which file it uses.

**One-off override:** that same Icon row also lets you upload an image directly for a single Dominion (downscaled and centre-cropped client-side, then stored inline on that Dominion as `img`, a data-URL). An upload always takes priority over both the convention file and an `icon` override, for that one Dominion only — handy for a Dominion that doesn't have official art yet, or a placeholder before you've prepared the real file.

**Reward tiers:** every Dominion belongs to a tier — Epic, Common, or Frontier for the 26 resource-bonus Dominions (matching the game's own "Dominion Schedule" screen), or Legendary for The Throne and the three Watchtowers (this tool's own addition, not an in-game tier, just giving the conquest points their own distinct colour instead of a flat generic one). `DOMINION_TIERS` and `DOMINION_INFO` near the top of `index.html`'s script hold each tier's colour and each Dominion's specific buff. A Dominion's own tile border and label default to its tier colour (still overridable per-marker via `d.color`, same as any other marker); its buff is shown as a second line under its name once you're zoomed in enough to read it, for the Dominions that have one — Legendary entries have a tier/colour but no buff line, since The Throne and Watchtowers aren't a resource bonus. The no-build *zone* around every Dominion is a flat colour (`DOMINION_ZONE_COLOR`) regardless of tier, deliberately — tier is still visible up close via the tile border, without 34 Dominions painting the whole map in four competing colours.

## Controls

| Action | Desktop | Mobile |
|---|---|---|
| Pan | Drag empty space (works in any tool) | Single-finger drag on empty space |
| Zoom | Scroll wheel | Pinch |
| Deselect back to Pan | Esc, or click the "🖐 Pan" button | Tap the "🖐 Pan" button |
| Place Hub/Outpost/City | Click empty hex with that tool active | Tap empty hex |
| Place a new Dominion (design mode only) | Click empty hex with Dominion tool active | Tap empty hex |
| Move a marker (Hub/Outpost/City/Dominion) | Click-drag the marker itself (any tool except Erase) | Touch-drag the marker |
| Rename / recolour a marker | Click the marker without dragging it (any tool except Erase) | Tap without dragging |
| Paint terrain (design mode only) | Click, or click-drag to paint a stroke, with Terrain tool active | Tap or drag |
| Erase | Click, or click-drag to erase a stroke, with Erase tool active | Tap or drag |
| Quick-erase terrain (design mode only) | Right-click, or right-click-drag | — (no touch equivalent) |

Markers can't be placed or dragged onto blocked terrain (see above) — the info bar explains why if a placement is refused.

**Hub-only Core restriction:** a Hub specifically can't be placed or dragged into the Core zone (Outposts and Cities have no such restriction). A Hub sitting just outside Core can still have its territory overlay legitimately reach into Core — this only restricts where the Hub marker itself can sit.

**Quick-erase:** while painting terrain, right-click-drag erases a stroke without switching off whatever brush you have selected — handy for undoing a mistake mid-stroke. It only ever erases terrain, never a Hub/Outpost/City, and only works in design mode; guildmates in the locked-down planner keep their browser's normal right-click menu.

## Dominions

Dominions are contested points worth a buff — a `DOM_R` (6-tile) radius around each one where **no Hub, Outpost, or City can be placed or dragged in**, regardless of terrain or zone. They're drawn as a purple hex badge with a purple-tinted no-build zone (colour changeable per-Dominion the same way as any other marker).

- **Placing a new one** is design-mode only (`?design`) — Dominions are canonical map features, not something an individual guildmate should be creating.
- **Dragging or renaming** is design-mode only *except* for a Dominion explicitly marked `"movable": true` in its JSON — currently just the three Watchtowers — which can be dragged/renamed in either mode, so anyone can reposition them after the Throne is recaptured without needing design-mode access. Everything else (the 15 fixed Dominions + the Throne) is locked to design-mode-only editing, so an ordinary guildmate can't accidentally nudge one.
- **Erasing one** is design-mode only, same as terrain — it's map data, not a per-guild placement.
- **Icons** work the same way as terrain images — drop a correctly-named PNG into `assets/dominions/` and it's picked up automatically, no code change or upload needed. No icon set yet just falls back to the plain coloured hex badge with a "D". See "Adding Dominion icons" below for the exact filenames and an alternative one-off upload option.
- **Reward tier** (Epic/Common/Frontier, or Legendary for The Throne/Watchtowers) drives a Dominion's default badge/territory colour and shows its buff under its name once zoomed in — see "Adding Dominion icons" below for how tiers and buffs are defined.

The current known Dominions (including the Throne and the three Watchtowers, whose positions shift each time the Throne is captured) are seeded in `map-data/default.json`. To make a *new* Dominion draggable outside design mode, add `"movable": true` to its entry there.

**Colour-coding a "war map":** placing or clicking a Hub/Outpost/City opens a dialog with a label field and a colour picker. Give different guilds/alliances their own colour and the marker, its label, and its territory overlay (Hub/Outpost) all pick it up — handy for showing contested or allied territory at a glance. Leaving the colour at its default keeps the normal blue/orange/gold scheme.

## Map data format

Exported JSON looks like this — safe to hand-edit:

```json
{
  "terrain": {
    "45,112": "deadMountains",
    "46,112": "deadMountains"
  },
  "hubs": [
    { "col": 129, "row": 80, "label": "Main Hub", "color": "#2a5aff" }
  ],
  "outposts": [
    { "col": 115, "row": 95, "label": "North OP", "color": "#e07800" }
  ],
  "cities": [
    { "col": 125, "row": 140, "label": "Capital", "color": "#d4a017" }
  ],
  "dominions": [
    { "col": 125, "row": 125, "label": "The Throne", "color": "#8e44ad" }
  ]
}
```

`color` is optional on every marker — omit it (or leave an older save without one) and it falls back to that type's default colour (`MARKER_DEFAULTS` in `index.html`). This is the same file shape used by both `map-data/default.json` and the "↓ Export"/"↑ Import" buttons.

Dominions can also carry an optional `img` field — a data-URL holding a small uploaded icon (see "Dominions" above). It's plain text (base64) so it still diffs/reviews fine, just a longer line per Dominion that has one.

## Map constants

Edit these at the top of the `<script>` block if the game patches zone sizes:

| Constant | Value | Meaning |
|---|---|---|
| `GRID` | 250 | Map is 250×250 hexes, columns/rows 0-249 |
| `CX / CY` | 125 | Centre coordinates |
| `CORE_R` | 56 | Core radius (innermost zone) |
| `MID_R` | 105 | Midlands outer radius |
| `HUB_R` | 10 | Guild hub territory radius |
| `OUT_R` | 5 | Outpost territory radius |
| `DOM_R` | 6 | Dominion no-build radius |

Zone order, innermost to outermost: **Core** (d ≤ `CORE_R`) → **Midlands** (d ≤ `MID_R`) → **Sanctuary** (everything beyond).

## Performance

Each unique (terrain-or-zone, zoom-level) hex is rendered once to a small offscreen canvas and reused for every hex of that kind, so panning/zooming blits cached bitmaps instead of redrawing every hex from scratch — this is what keeps scrolling smooth even with terrain images loaded. Rendering during pan/zoom/drag is also throttled to one frame per animation frame via `requestAnimationFrame`. At extreme zoom-out (viewing most of the 250×250 grid at once) rendering thousands of hexes is inherently heavier — that's an expected limit of hex-by-hex rendering, not a bug.
