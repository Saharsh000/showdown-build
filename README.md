# 🎮 Pokémon Showdown Build Guide

> A comprehensive guide to building a Node.js Pokémon battle bot using the Pokémon Showdown engine

---

## 📋 Table of Contents

1. [Package Structure & Imports](#1-package-structure--imports--exports)
2. [BattleStream Mechanics](#2-how-battlestream-works)
3. [Building Teams](#3-building-and-packing-custom-teams)
4. [Pokédex Access](#4-accessing-the-pokédex)
5. [Battle Protocol](#5-the-battle-protocol-explained)
6. [Bot Creation](#6-using-battleplayer-and-randomplayer-ai)
7. [Format Strings](#7-available-battle-format-strings)
8. [Complete Example](#8-end-to-end-working-example)

---

## 1️⃣ Package Structure & Imports & Exports

### 🌍 Ecosystem Overview: `pokemon-showdown` vs `@pkmn/sim`

There are two primary ways to access the Pokémon Showdown engine in Node.js:

#### ✅ `@pkmn/sim` (Recommended for npm development)

- Maintained by core Pokémon Showdown / Smogon contributors (under the `@pkmn` organization)
- Officially maintained, typed, and versioned extraction of the `sim/` directory
- Designed specifically to be imported as an npm library

**Installation:**
```bash
npm install @pkmn/sim
```

**Imports:**
```javascript
// CommonJS
const { BattleStreams, Dex, Teams, RandomPlayerAI, Battle } = require('@pkmn/sim');

// ES Module / TypeScript
import { BattleStreams, Dex, Teams, RandomPlayerAI, Battle } from '@pkmn/sim';
```

#### 📦 `pokemon-showdown` (The monolithic server repo)

- Published on npm and hosted at [smogon/pokemon-showdown](https://github.com/smogon/pokemon-showdown)
- Designed primarily as a standalone server application
- Main entry point: `"main": "dist/sim/index.js"`

**Imports:**
```javascript
// CommonJS from pokemon-showdown:
const { BattleStream, BattlePlayer } = require('pokemon-showdown/dist/sim/battle-stream');
const { Dex, toID } = require('pokemon-showdown/dist/sim/dex');
const { Teams } = require('pokemon-showdown/dist/sim/teams');
const { RandomPlayerAI } = require('pokemon-showdown/dist/sim/tools/random-player-ai');

// Or using the top-level main entrypoint:
const Sim = require('pokemon-showdown');
const stream = new Sim.BattleStream();
```

### 📂 Key Files in `sim/`

| File | Purpose |
|------|---------|
| `sim/battle-stream.ts` | Defines `BattleStream`, `BattleTextStream`, and `BattlePlayer` |
| `sim/battle.ts` | Core Battle class managing game state, turns, RNG, mechanics |
| `sim/dex.ts` | Core Dex data layer (species, moves, abilities, items, rulesets) |
| `sim/teams.ts` | Teams class for parsing, packing, unpacking, and exporting team sets |
| `sim/tools/random-player-ai.ts` | Example `RandomPlayerAI` class extending `BattlePlayer` |
| `sim/prng.ts` | Seedable pseudo-random number generator (PRNG) |

---

## 2️⃣ How BattleStream Works: Starting Battles, Sending Moves, Reading Output

`BattleStream` is an `ObjectReadWriteStream<string>`. You interact with it by writing command strings starting with `>` and reading pipe-delimited protocol chunks.

### 🔀 Option A: Direct BattleStream (Single Stream)

```javascript
const { BattleStream } = require('@pkmn/sim');
const stream = new BattleStream();

// 1. Read battle output asynchronously
(async () => {
  for await (const chunk of stream) {
    const lines = chunk.split('\n');
    for (const line of lines) {
      console.log('OUTPUT:', line);
    }
  }
})();

// 2. Start battle and assign players
stream.write(`>start {"formatid":"gen9ou"}\n`);
stream.write(`>player p1 {"name":"Alice","team":"${p1PackedTeam}"}\n`);
stream.write(`>player p2 {"name":"Bob","team":"${p2PackedTeam}"}\n`);

// 3. Send player choices
stream.write(`>p1 move 1\n`);           // Move
stream.write(`>p1 move 1 terastallize\n`); // Terastallization
stream.write(`>p1 switch 3\n`);         // Switch
stream.write(`>p1 team 123456\n`);      // Team preview
```

### 👥 Option B: Split Streams (`BattleStreams.getPlayerStreams`)

Splits one `BattleStream` into dedicated player streams:

```javascript
const { BattleStreams } = require('@pkmn/sim');
const battleStream = new BattleStreams.BattleStream();
const streams = BattleStreams.getPlayerStreams(battleStream);

// streams.p1 - Player 1 channel (p1's view only)
// streams.p2 - Player 2 channel (p2's view only)
// streams.omniscient - Spectator/admin stream (all events)

// Listen to the full battle log
(async () => {
  for await (const chunk of streams.omniscient) {
    console.log('[SPECTATOR]', chunk);
  }
})();

// Initialize battle on omniscient stream
streams.omniscient.write(`>start {"formatid":"gen9ou"}\n`);
streams.omniscient.write(`>player p1 {"name":"Bot1","team":"${p1Packed}"}\n`);
streams.omniscient.write(`>player p2 {"name":"Bot2","team":"${p2Packed}"}\n`);

// Send choices directly on player streams:
streams.p1.write('move 1');
streams.p2.write('switch 2');
```

### 🎯 Choice Command Reference

| Command | Usage |
|---------|-------|
| `move 1` | Use move in slot 1 (1-indexed, or use name: `move thunderbolt`) |
| `move 1 terastallize` | Use move 1 and Terastallize (Gen 9) |
| `move 1 mega` | Use move 1 and Mega Evolve (Gen 6/7) |
| `move 1 dynamax` | Use move 1 and Dynamax (Gen 8) |
| `move 1 1` | Doubles: move slot 1, target right opponent |
| `switch 2` | Switch to Pokémon at slot 2 |
| `team 123456` | Set team order during team preview |
| `default` | Simulator chooses default valid move or lead |

---

## 3️⃣ Building and Packing Custom Teams Programmatically

Pokémon Showdown uses three team representations:

- **Human-Readable Format**: Standard Pokémon Showdown export format
- **JSON Format** (`PokemonSet[]`): Array of set objects used internally
- **Packed Format**: Minified pipe-separated string for `>player ... "team":""...`

### 📚 Teams API

| Method | Description |
|--------|-------------|
| `Teams.pack(team: PokemonSet[]): string` | Pack array of sets into minified string |
| `Teams.unpack(packed: string): PokemonSet[]` | Decode packed string into sets |
| `Teams.import(text: string): PokemonSet[]` | Parse human-readable export text |
| `Teams.export(team: PokemonSet[]): string` | Convert sets to human-readable format |
| `Teams.generate(formatid: string): PokemonSet[]` | Generate random team |

### 💾 Option 1: Convert Export String to Packed

```javascript
const { Teams } = require('@pkmn/sim');

const exportText = `
Dragapult @ Choice Specs
Ability: Infiltrator
Tera Type: Ghost
EVs: 252 SpA / 4 SpD / 252 Spe
Timid Nature
- Shadow Ball
- Draco Meteor
- U-turn
- Thunderbolt

Kingambit @ Leftovers
Ability: Supreme Overlord
Tera Type: Flying
EVs: 212 HP / 252 Atk / 44 Spe
Adamant Nature
- Kowtow Cleave
- Sucker Punch
- Iron Head
- Swords Dance
`;

const teamSets = Teams.import(exportText);
const packedTeam = Teams.pack(teamSets);
console.log(packedTeam);
// Output: Dragapult||choicespecs|infiltrator|shadowball,dracometeor,uturn,thunderbolt|Timid|...
```

### 🛠️ Option 2: Programmatically Construct `PokemonSet[]`

```javascript
const myTeam = [
  {
    name: 'Dragapult',
    species: 'Dragapult',
    item: 'Choice Specs',
    ability: 'Infiltrator',
    moves: ['Shadow Ball', 'Draco Meteor', 'U-turn', 'Thunderbolt'],
    nature: 'Timid',
    gender: 'M',
    evs: { hp: 0, atk: 0, def: 0, spa: 252, spd: 4, spe: 252 },
    ivs: { hp: 31, atk: 31, def: 31, spa: 31, spd: 31, spe: 31 },
    level: 100,
    shiny: false,
    teraType: 'Ghost',
  },
  {
    name: 'Gholdengo',
    species: 'Gholdengo',
    item: 'Covert Cloak',
    ability: 'Good as Gold',
    moves: ['Make It Rain', 'Shadow Ball', 'Nasty Plot', 'Recover'],
    nature: 'Modest',
    evs: { hp: 252, atk: 0, def: 0, spa: 252, spd: 4, spe: 0 },
    level: 100,
    teraType: 'Fighting',
  }
];

const packed = Teams.pack(myTeam);
```

### 🔧 Packed Format Field Structure

Each Pokémon is encoded with pipe (`|`) delimiters:

```
name|species|item|ability|moves|nature|evs|gender|ivs|shiny|level|happiness,pokeball,hpType,gigantamax,dynamaxLevel,teraType
```

Individual Pokémon are separated by `]`.

---

## 4️⃣ Accessing the Pokédex (Dex)

The `Dex` module provides the entire Showdown database. By default, it's configured for the latest generation (Gen 9).

### 🔍 Basic Queries

```javascript
const { Dex, toID } = require('@pkmn/sim');

// 1. Look up Pokémon Species
const pikachu = Dex.species.get('pikachu');
console.log(pikachu.name);           // "Pikachu"
console.log(pikachu.num);            // 25
console.log(pikachu.types);          // ["Electric"]
console.log(pikachu.baseStats);      // { hp: 35, atk: 55, def: 40, spa: 50, spd: 50, spe: 90 }
console.log(pikachu.abilities);      // { '0': 'Static', 'H': 'Lightning Rod' }
console.log(pikachu.tier);           // "LC"
console.log(pikachu.prevo);          // "Pichu"
console.log(pikachu.evos);           // ["Raichu", "Raichu-Alola"]
console.log(pikachu.exists);         // true

// Formes
const rotomWash = Dex.species.get('Rotom-Wash');
console.log(rotomWash.types);        // ["Electric", "Water"]
console.log(rotomWash.baseSpecies);  // "Rotom"

// 2. Look up Abilities
const intimidate = Dex.abilities.get('Intimidate');
console.log(intimidate.name);        // "Intimidate"
console.log(intimidate.desc);        // Full description
console.log(intimidate.rating);      // Competitive viability rating

// 3. Look up Moves
const thunderbolt = Dex.moves.get('Thunderbolt');
console.log(thunderbolt.basePower);  // 90
console.log(thunderbolt.type);       // "Electric"
console.log(thunderbolt.category);   // "Special"
console.log(thunderbolt.accuracy);   // 100
console.log(thunderbolt.pp);         // 15

// 4. Look up Items
const leftovers = Dex.items.get('Leftovers');
console.log(leftovers.name);         // "Leftovers"

// 5. Look up Natures & Types
const timid = Dex.natures.get('Timid');
console.log(timid.plus);             // 'spe'
console.log(timid.minus);            // 'atk'
```

### 📖 Looking Up Learnsets

```javascript
// Get raw learnset data
const learnsetData = Dex.species.getLearnsetData(pikachu.id);
// Maps move ID to learning sources:
// {
//   thunderbolt: ['9M', '8M', '7M', ...], // '9M' = Gen 9 TM
//   volttackle: ['9E', '8E', ...],        // '9E' = Egg move
//   thundershock: ['9L1', '8L1', ...]     // '9L1' = Gen 9 Level 1
// }

// Get full array of all moves the species can legally learn
const allMoves = Dex.species.getFullLearnset(pikachu.id);
console.log(allMoves.includes('thunderbolt')); // true
```

### 🕐 Generation-Specific Data

```javascript
// Access Gen 1 rules and mechanics:
const gen1Dex = Dex.mod('gen1');
const gen1Tackle = gen1Dex.moves.get('Tackle');
console.log(gen1Tackle.basePower); // 35 (was 35 in Gen 1)

const gen8Dex = Dex.forGen(8);
const zacian = gen8Dex.species.get('Zacian-Crowned');
console.log(zacian.baseStats.atk); // 170 (nerfed to 150 in Gen 9)
```

---

## 5️⃣ The Battle Protocol Explained

All messages sent from the simulator are newline-separated, pipe-delimited strings: `|command|arg1|arg2|....`

### 📡 Common Protocol Messages

#### `|turn|TURN_NUMBER`
Signals the start of a new turn.
```
|turn|1
```

#### `|switch|POKEMON|DETAILS|HP STATUS`
Signals a Pokémon entering the field.
```
|switch|p1a: Dragapult|Dragapult, L100, M|280/280
|switch|p2a: Ting-Lu|Ting-Lu, L100|450/450
```

#### `|move|POKEMON|MOVE|TARGET|[flags]`
Signals a Pokémon executing an attack.
```
|move|p1a: Dragapult|Shadow Ball|p2a: Ting-Lu
|move|p2a: Cinderace|Pyro Ball|p1a: Corviknight|[miss]
```

#### `|-damage|POKEMON|HP STATUS|[from]SOURCE`
Signals HP reduction.
```
|-damage|p2a: Ting-Lu|380/450
|-damage|p1a: Dragapult|252/280|[from] item: Life Orb
|-damage|p1a: Dragapult|0 fnt
|faint|p1a: Dragapult
```

#### `|request|JSON_STRING` ⭐
**Most critical message for bots!** Sent when the simulator requires input.

```json
{
  "rqid": 1,
  "active": [
    {
      "moves": [
        {
          "move": "Shadow Ball",
          "id": "shadowball",
          "pp": 24,
          "maxpp": 24,
          "target": "normal",
          "disabled": false
        }
      ],
      "canTerastallize": "Ghost",
      "trapped": false
    }
  ],
  "side": {
    "name": "Bot 1",
    "id": "p1",
    "pokemon": [
      {
        "ident": "p1: Dragapult",
        "details": "Dragapult, L100, M",
        "condition": "280/280",
        "active": true,
        "stats": { "atk": 256, "def": 186, "spa": 299, "spd": 186, "spe": 421 },
        "moves": ["shadowball", "dracometeor", "uturn", "thunderbolt"],
        "ability": "infiltrator",
        "item": "choicespecs",
        "teraType": "Ghost"
      }
    ]
  }
}
```

### 🎛️ Request Scenarios to Handle

| Scenario | Action |
|----------|--------|
| `request.wait === true` | Opponent deciding or simultaneous switch pending — **DO NOT** send choice |
| `request.teamPreview === true` | Battle start — send `team 123456` or `default` |
| `request.forceSwitch` | Active Pokémon fainted — **must switch** with `switch <index>` |
| `request.active` | Standard turn — choose move with `move <slot>` or switch |

---

## 6️⃣ Using BattlePlayer and RandomPlayerAI

### 🤖 How BattlePlayer Works Internally

`BattlePlayer` is the abstract base class that:
- Reads string chunks from `this.playerStream`
- Detects `|request|` lines and parses JSON, calling `this.receiveRequest(request)`
- Detects `|error|` lines and calls `this.receiveError(error)`
- Provides `this.choose(choiceString)` to write actions back

### 🎲 Using Built-in RandomPlayerAI

```javascript
const { BattleStreams, RandomPlayerAI, Teams } = require('@pkmn/sim');

const battleStream = new BattleStreams.BattleStream();
const streams = BattleStreams.getPlayerStreams(battleStream);

// Initialize two RandomPlayerAI bots
const p1Bot = new RandomPlayerAI(streams.p1);
const p2Bot = new RandomPlayerAI(streams.p2);

// Start bot listening loops
void p1Bot.start();
void p2Bot.start();

// Configure battle
const p1Team = Teams.pack(Teams.generate('gen9randombattle'));
const p2Team = Teams.pack(Teams.generate('gen9randombattle'));

streams.omniscient.write(`>start {"formatid":"gen9randombattle"}\n`);
streams.omniscient.write(`>player p1 {"name":"Alice","team":"${p1Team}"}\n`);
streams.omniscient.write(`>player p2 {"name":"Bob","team":"${p2Team}"}\n`);
```

### 💡 Writing a Custom AI by Extending BattlePlayer

```javascript
const { BattlePlayer } = require('@pkmn/sim');

class SmartBattleBot extends BattlePlayer {
  constructor(playerStream, debug = false) {
    super(playerStream, debug);
  }

  receiveRequest(request) {
    // 1. Wait flag
    if (request.wait) return;

    // 2. Team Preview
    if (request.teamPreview) {
      this.choose('default');
      return;
    }

    // 3. Forced Switch
    if (request.forceSwitch) {
      const validSwitchSlots = [];
      request.side.pokemon.forEach((p, index) => {
        if (!p.active && !p.condition.endsWith('fnt')) {
          validSwitchSlots.push(index + 1);
        }
      });
      if (validSwitchSlots.length > 0) {
        this.choose(`switch ${validSwitchSlots[0]}`);
      } else {
        this.choose('default');
      }
      return;
    }

    // 4. Standard Turn
    if (request.active) {
      const activePokemon = request.active[0];
      const availableMoves = [];
      
      activePokemon.moves.forEach((m, index) => {
        if (!m.disabled && m.pp > 0) {
          availableMoves.push(index + 1);
        }
      });

      if (availableMoves.length > 0) {
        const teraSuffix = activePokemon.canTerastallize ? ' terastallize' : '';
        this.choose(`move ${availableMoves[0]}${teraSuffix}`);
      } else {
        this.choose('move 1');
      }
      return;
    }

    this.choose('default');
  }

  receiveError(error) {
    console.error(`[${this.constructor.name}] Battle Error:`, error.message);
  }
}

module.exports = { SmartBattleBot };
```

---

## 7️⃣ Available Battle Format Strings

Format IDs are normalized via `toID()` (lowercase, alphanumeric only).

### 🏆 Common Gen 9 Formats

**Standard Singles:**
- `gen9ou` – OverUsed (flagship)
- `gen9ubers` – Ubers
- `gen9uu` – UnderUsed
- `gen9ru` – RarelyUsed
- `gen9nu` – NeverUsed
- `gen9pu` – PU
- `gen9lc` – Little Cup (Level 5 unevolved)
- `gen9monotype` – Monotype (all Pokémon same type)
- `gen9anythinggoes` – Anything Goes (no banlist)
- `gen91v1` – 1v1 Singles
- `gen9customgame` – Custom Game (no restrictions)

**Random Battles:**
- `gen9randombattle` – Standard competitive random
- `gen9randomdoublesbattle` – Doubles random
- `gen9monotyperandombattle` – Monotype random
- `gen9hackmonscup` – Randomized moves/abilities

**Doubles & VGC:**
- `gen9doublesou` – Doubles OverUsed
- `gen9doublesubers` – Doubles Ubers
- `gen9vgc2024regh` / `gen9vgc2025...` – Official VGC rules
- `gen9doublescustomgame` – Doubles Custom

**Past Generations:** Replace `gen9` with `gen8`, `gen7`, `gen6`, etc.

### 🔎 Querying Formats Programmatically

```javascript
const { Dex } = require('@pkmn/sim');

// Get all registered formats
const allFormats = Dex.formats.all();
allFormats.forEach(f => {
  if (f.id.startsWith('gen9')) {
    console.log(f.id, '-', f.name);
  }
});

// Inspect a specific format
const format = Dex.formats.get('gen9ou');
console.log('Name:', format.name);         // "[Gen 9] OU"
console.log('Game Type:', format.gameType); // "singles"
console.log('Ruleset:', format.ruleset);   // ['Standard', 'Sleep Moves Clause', ...]
console.log('Banlist:', format.banlist);   // ['Uber', 'AG', 'Arena Trap', ...]
```

---

## 8️⃣ End-to-End Working Example: Full Battle Simulation

Here is a complete, standalone script demonstrating how to run a simulated match:

```javascript
const { BattleStreams, RandomPlayerAI, Teams } = require('@pkmn/sim');
const { SmartBattleBot } = require('./smart-bot');

async function runBattle() {
  // 1. Initialize the BattleStream and Player Streams
  const battleStream = new BattleStreams.BattleStream();
  const streams = BattleStreams.getPlayerStreams(battleStream);

  // 2. Instantiate players
  const p1 = new SmartBattleBot(streams.p1);
  const p2 = new RandomPlayerAI(streams.p2);

  // 3. Start player event loops
  void p1.start();
  void p2.start();

  // 4. Listen to battle events via the spectator stream
  (async () => {
    for await (const chunk of streams.omniscient) {
      const lines = chunk.split('\n');
      for (const line of lines) {
        if (line.startsWith('|turn|')) {
          console.log(`--- TURN ${line.split('|')[2]} ---`);
        } else if (line.startsWith('|move|')) {
          const [, , pokemon, move, target] = line.split('|');
          console.log(`${pokemon} used ${move}!`);
        } else if (line.startsWith('|switch|')) {
          const [, , pokemon, details] = line.split('|');
          console.log(`${pokemon} switched in (${details})`);
        } else if (line.startsWith('|faint|')) {
          const [, , pokemon] = line.split('|');
          console.log(`${pokemon} fainted!`);
        } else if (line.startsWith('|win|')) {
          const [, , winner] = line.split('|');
          console.log(`🏆 Winner: ${winner}`);
        }
      }
    }
  })();

  // 5. Generate or import teams
  const p1Team = Teams.pack(Teams.generate('gen9randombattle'));
  const p2Team = Teams.pack(Teams.generate('gen9randombattle'));

  // 6. Launch the battle
  await streams.omniscient.write(`>start {"formatid":"gen9randombattle"}\n`);
  await streams.omniscient.write(`>player p1 {"name":"SmartBot","team":"${p1Team}"}\n`);
  await streams.omniscient.write(`>player p2 {"name":"RandomAI","team":"${p2Team}"}\n`);
}

runBattle().catch(console.error);
```

---

## 🚀 Quick Start

1. **Install the package:**
   ```bash
   npm install @pkmn/sim
   ```

2. **Create your bot:**
   ```bash
   cp the SmartBattleBot template from Section 6
   ```

3. **Run a battle:**
   ```bash
   node run-battle.js
   ```

---

## 📚 Resources

- [Pokémon Showdown GitHub](https://github.com/smogon/pokemon-showdown)
- [@pkmn/sim NPM](https://www.npmjs.com/package/@pkmn/sim)
- [Smogon University](https://www.smogon.com/)

---

**Happy building! 🎮⚡**