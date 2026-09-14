# Pokémon Battle Bot — Single-File Discord Bot

Build a fully self-contained Discord bot (`bot.js`) that lets users battle AI opponents with Pokémon chosen from the existing sprite library, using the `pokemon-showdown` package as the battle simulator and `canvas` for rendering battle images with the project's backgrounds, UI overlays, and animated sprite assets.

## Proposed Changes

### [NEW] [bot.js](file:///Users/saharsh/Downloads/pokeventure-image-main-2/bot.js)

A single ~1,200-line Node.js file containing everything:

#### 1. Sprite Scanner & Pokémon Pool Builder
- On startup, scans `src/img/front/` and `src/img/back/` directories
- Extracts base Pokémon names from filenames (stripping extensions, normalizing forms)
- Maps each sprite name → Showdown species ID (e.g. `charizard-megax.gif` → `charizardmegax`)
- Validates against the `pokemon-showdown` Dex — only Pokémon recognized by the simulator are added to the battle pool
- **Event sprites** (halloween, xmas, bday, easter, vday) are tracked separately as cosmetic "skins" — they use the base Pokémon's stats/moves but display the themed sprite
- **Digimon** sprites (241 files) are catalogued but excluded from the competitive pool (no Showdown data). They appear in a showcase/gallery command
- **Custom Megas** (e.g. `dragonite-mega`, `darkrai-mega`) are catalogued for display but use the base Pokémon's data for battles
- **Result**: A pool of ~800–1000 battle-ready Pokémon covering Gens 1–9

#### 2. Team Generator
- For each battle, randomly picks 6 Pokémon from the validated pool (no duplicates)
- For each Pokémon, pulls from the Showdown Dex:
  - **Stats**: Base stats from `Dex.species.get()`
  - **Ability**: Random valid ability from the species data
  - **Moves**: 4 random moves from the species' learnset (filtered to damaging + status mix)
  - **Item**: Random competitive item from a curated list (Life Orb, Choice Scarf, Leftovers, etc.)
  - **Nature**: Random nature
  - **EVs**: Random 252/252/4 spread
  - **IVs**: All 31
  - **Level**: 100
- Packs team using `Teams.pack()` for the battle stream
- Has a ~20% chance per Pokémon to use an event skin (Halloween/Xmas/etc.) if one exists for that species

#### 3. Battle Engine (Showdown Integration)
- Creates a `BattleStream` per active battle
- Format: `gen9anythinggoes` (allows all Pokémon including megas, legendaries, etc.)
- Parses the Showdown protocol line-by-line:
  - `|request|{...}` → Extracts active Pokémon info, available moves, team status
  - `|switch|` → Tracks which Pokémon is on the field
  - `|move|`, `|-damage|`, `|-heal|`, `|-status|` → Builds a battle log
  - `|faint|` → Handles faints and forced switches
  - `|win|` → Ends the battle
- **AI opponent**: Picks moves using simple heuristics (random with slight preference for super-effective moves, and switches when HP is critical)
- Stores active battles in a `Map<channelId, BattleState>`

#### 4. Image Renderer (Canvas)
- Replicates the existing PHP renderer's layout on a **518×288** canvas
- Uses `registerFont()` to load `pokemon_fire_red.ttf` for authentic text
- Layer order:
  1. Random background from `src/img/bgs/` (19 unique JPEG backgrounds: `-1` through `20`)
  2. Enemy front sprite from `src/img/front/` (or `front-shiny/`)
  3. Player back sprite from `src/img/back/` (or `back-shiny/`)
  4. UI overlay from `src/img/ui.png`
  5. HP bars (green > yellow > red based on %) 
  6. Gender icons (`male.png` / `female.png`)
  7. Name text, level, and HP numbers
- GIF sprites loaded via `loadImage()` (renders first frame — fine for static battle snapshots)
- Falls back to `missingno.png` for any missing sprites
- Exports as PNG buffer → `AttachmentBuilder` for Discord

#### 5. Discord Bot Interface (discord.js v14)

**Slash Commands:**

| Command | Description |
|---|---|
| `/battle` | Start a new battle against AI with random teams |
| `/battle-theme <theme>` | Start a themed battle (halloween, xmas, bday, easter, vday) — forces event skins for all Pokémon that have them |
| `/moves` | Show your current Pokémon's moves (if mid-battle) |
| `/team` | View your full team and their HP status |
| `/forfeit` | End the current battle |
| `/pokedex` | Browse available Pokémon with sprite previews |
| `/gallery <category>` | Browse sprites by category: event, digimon, mega, gmax, regional |

**Battle Flow (Button-Based):**
1. User types `/battle` → Bot generates both teams, picks a random background
2. Bot sends an embed with the rendered battle image + 4 move buttons + a "Switch" button
3. User clicks a move → AI picks its move → turn resolves
4. Bot sends updated battle image with a battle log summary of what happened
5. On faint → forced switch menu (select menu with remaining alive Pokémon)
6. On win/loss → victory/defeat embed with final stats

**Interactive Elements:**
- **Move buttons**: 4 `ButtonBuilder` buttons labeled with move names + PP, color-coded by type
- **Switch button**: Opens a `StringSelectMenuBuilder` with team Pokémon (name, HP%, status)
- **Mega Evolution**: Extra toggle button when a Mega-capable Pokémon is active
- 60-second timeout per turn (auto-selects random move)

#### 6. Pokédex / Gallery System
- `/pokedex` renders a paginated embed showing Pokémon sprites from `src/img/front/`
- `/gallery event` shows all Halloween, Xmas, Birthday, Easter, Valentine's sprites
- `/gallery digimon` shows all 241 Digimon sprites
- `/gallery mega` shows all Mega Evolution sprites (official + custom)
- Each page renders a 4×3 grid of sprites on a canvas with labels

---

### [NEW] [package.json](file:///Users/saharsh/Downloads/pokeventure-image-main-2/package.json)

```json
{
  "name": "pokeventure-battle-bot",
  "version": "1.0.0",
  "description": "Pokemon Battle Discord Bot with AI opponents",
  "main": "bot.js",
  "scripts": {
    "start": "node bot.js",
    "register": "node bot.js --register"
  },
  "dependencies": {
    "discord.js": "^14.16.0",
    "pokemon-showdown": "^0.11.9",
    "canvas": "^2.11.2",
    "dotenv": "^16.4.0"
  }
}
```

> [!NOTE]
> The `canvas` package requires system libraries (Cairo, Pango, libgif, libjpeg). On macOS: `brew install pkg-config cairo pango libpng jpeg giflib librsvg`. On Ubuntu: `sudo apt-get install build-essential libcairo2-dev libpango1.0-dev libjpeg-dev libgif-dev librsvg2-dev`. If system deps are problematic, `@napi-rs/canvas` is a drop-in zero-compile replacement.

---

### [NEW] [.env.example](file:///Users/saharsh/Downloads/pokeventure-image-main-2/.env.example)

```
DISCORD_TOKEN=your_bot_token_here
CLIENT_ID=your_application_client_id_here
GUILD_ID=your_test_server_id_here
```

---

## Key Design Decisions

### Why `gen9anythinggoes` instead of `gen9randombattle`?
Random Battle auto-generates teams which means we can't control which Pokémon appear — defeating the purpose of using our sprite library. Anything Goes allows all Pokémon (including legendaries, megas, banned stuff) so we can build teams from our full sprite pool.

### Why event sprites as "skins" instead of separate Pokémon?
Event sprites (halloween Pikachu, xmas Bulbasaur, etc.) don't have separate competitive data in Showdown. They use the base Pokémon's stats, moves, and abilities — only the visual appearance changes. This lets us showcase all ~200 event sprites without needing custom stat data.

### How are sprites mapped to Showdown species?
The filename normalization follows the same logic as the existing PHP `normalizeName()`:
- Lowercase the name
- Strip special characters (spaces, apostrophes, periods, colons, accents)
- Handle special cases (mega-x/y, regional forms, nidoran gender, porygon hyphens)
- Event suffixes (-halloween, -xmas, -bday, -easter, -vday) are stripped to find the base species

### Why not use the Digimon for battles?
The 241 Digimon sprites have no corresponding data in Showdown's Dex (no stats, moves, abilities, types). They're included in the gallery for browsing and could be added as joke battle Pokémon in a future update (mapping them to random existing stats).

---

## Verification Plan

### Automated Tests
```bash
# Install dependencies
npm install

# Register slash commands with Discord
node bot.js --register

# Start the bot
node bot.js
```

### Manual Verification
1. **Sprite scanning**: Bot logs on startup how many Pokémon were found in each category (battle pool, event skins, digimon, custom megas)
2. **Battle flow**: Start a `/battle`, verify image renders correctly with background + sprites + UI + HP bars
3. **Move selection**: Click move buttons, verify turn resolves and image updates
4. **Switching**: Trigger a faint, verify switch menu appears with correct team info
5. **Event theme**: Use `/battle-theme halloween`, verify Halloween sprites are displayed
6. **Gallery**: Use `/gallery digimon`, verify Digimon sprites render in grid
7. **Edge cases**: Test with Pokémon that only have PNG sprites (no GIF), verify fallback to missingno for missing sprites
