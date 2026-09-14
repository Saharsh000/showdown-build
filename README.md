Comprehensive Guide: Using Pokémon Showdown Programmatically in Node.js
This report covers everything required to build a Node.js Pokémon battle bot using the Pokémon Showdown engine.
The repo will not contain any kind of code and it will just be this .md file
1. Package Structure, Imports, and Exports
Ecosystem Overview: pokemon-showdown vs @pkmn/sim
There are two primary ways to access the Pokémon Showdown engine in Node.js:

@pkmn/sim (Recommended for npm development)
Maintained by core Pokémon Showdown / Smogon contributors (under the @pkmn organization).
It is an officially maintained, typed, versioned extraction of the sim/ directory from smogon/pokemon-showdown.
Designed specifically to be imported as an npm library:
bash

npm install @pkmn/sim
Exports all simulation components from a clean top-level barrel:
javascript

// CommonJS
const { BattleStreams, Dex, Teams, RandomPlayerAI, Battle } = require('@pkmn/sim');
// ES Module / TypeScript
import { BattleStreams, Dex, Teams, RandomPlayerAI, Battle } from '@pkmn/sim';
pokemon-showdown (The monolithic server repo)
Published on npm as pokemon-showdown and hosted at smogon/pokemon-showdown on GitHub.
Designed primarily as a standalone server application rather than a client library.
Its package.json specifies:
json

"main": "dist/sim/index.js"
In smogon/pokemon-showdown, TypeScript source files live in sim/ and compile to dist/sim/ via node build (executed on postinstall).
If using pokemon-showdown directly, imports follow:
javascript

// CommonJS from pokemon-showdown:
const { BattleStream, BattlePlayer } = require('pokemon-showdown/dist/sim/battle-stream');
const { Dex, toID } = require('pokemon-showdown/dist/sim/dex');
const { Teams } = require('pokemon-showdown/dist/sim/teams');
const { RandomPlayerAI } = require('pokemon-showdown/dist/sim/tools/random-player-ai');
// Or using the top-level main entrypoint:
const Sim = require('pokemon-showdown');
const stream = new Sim.BattleStream();
const dex = Sim.Dex;
const teams = Sim.Teams;
Key Files in sim/
sim/battle-stream.ts – Defines BattleStream (ObjectReadWriteStream<string>), BattleTextStream, and BattlePlayer.
sim/battle.ts – Core Battle class managing game state, turns, RNG, mechanics.
sim/dex.ts – Core Dex data layer (access to species, moves, abilities, items, rulesets).
sim/teams.ts – Teams class for parsing, packing, unpacking, and exporting team sets.
sim/tools/random-player-ai.ts – Example RandomPlayerAI class that extends BattlePlayer.
sim/prng.ts – Seedable pseudo-random number generator (PRNG).
2. How BattleStream Works: Starting Battles, Sending Moves, Reading Output
BattleStream is an ObjectReadWriteStream<string>. You interact with it by writing command strings starting with > and reading pipe-delimited protocol chunks.

Option A: Direct BattleStream (Single Stream)
javascript

const { BattleStream } = require('@pkmn/sim'); // or require('pokemon-showdown/dist/sim/battle-stream')
const stream = new BattleStream();
// 1. Read battle output asynchronously
(async () => {
  for await (const chunk of stream) {
    // chunk is a multi-line string of protocol messages
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
// Singles move:
stream.write(`>p1 move 1\n`);
stream.write(`>p2 move 2\n`);
// Terastallization:
stream.write(`>p1 move 1 terastallize\n`);
// Switching:
stream.write(`>p1 switch 3\n`);
// Team preview selection (order of pokemon, or default):
stream.write(`>p1 team 123456\n`); // or `>p1 default\n`
Option B: Split Streams (BattleStreams.getPlayerStreams)
BattleStreams.getPlayerStreams(battleStream) splits one BattleStream into dedicated player streams and an omniscient stream:

streams.p1: Player 1 channel (receives only p1's view + |request| JSON for p1). Choices written to streams.p1 do not need the >p1 prefix (write move 1 or switch 2).
streams.p2: Player 2 channel.
streams.omniscient: Spectator/admin stream (receives all battle events; write >start and >player here).
javascript

const { BattleStreams } = require('@pkmn/sim');
const battleStream = new BattleStreams.BattleStream();
const streams = BattleStreams.getPlayerStreams(battleStream);
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
// Sending choices directly on player streams:
streams.p1.write('move 1');
streams.p2.write('switch 2');
Choice Command Reference:
Command Syntax	Usage
move 1	Use move in slot 1 (1-indexed). Can also use move name: move thunderbolt
move 1 terastallize	Use move 1 and Terastallize (Gen 9)
move 1 mega	Use move 1 and Mega Evolve (Gen 6/7)
move 1 dynamax	Use move 1 and Dynamax (Gen 8)
move 1 1 / move 1 -1	In Doubles: move <slot> <target>. Targets: 1=right opponent, 2=left opponent, -1=partner
switch 2	Switch to Pokémon at slot 2 in party
team 123456	Set team order during team preview
default	Simulator chooses default valid move or lead
3. Building and Packing Custom Teams Programmatically
Pokémon Showdown uses three team representations:

Human-Readable / Export Format: The standard Pokémon Showdown export format.
JSON Format (PokemonSet[]): Array of set objects used internally in JS/TS.
Packed Format: Minified pipe-separated string required by >player ... "team":"...".
The Teams API:
Teams.pack(team: PokemonSet[]): string -> Packs array of sets into a packed string.
Teams.unpack(packed: string): PokemonSet[] -> Decodes packed string into set array.
Teams.import(text: string): PokemonSet[] -> Parses human-readable export text.
Teams.export(team: PokemonSet[]): string -> Converts sets to human-readable export text.
Teams.generate(formatid: string): PokemonSet[] -> Generates random team (for randombattle formats).
Option 1: Convert Showdown Export String to Packed
javascript

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
// Output: Dragapult||choicespecs|infiltrator|shadowball,dracometeor,uturn,thunderbolt|Timid|,,,252,4,252|||||Ghost]Kingambit||leftovers|supremeoverlord|kowtowcleave,suckerpunch,ironhead,swordsdance|Adamant|212,252,,,44|||||Flying
Option 2: Programmatically Construct PokemonSet[]
javascript

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
Packed Format Field Structure
Each Pokémon is encoded with pipe | delimiters:

text

name|species|item|ability|moves|nature|evs|gender|ivs|shiny|level|happiness,pokeball,hpType,gigantamax,dynamaxLevel,teraType
Individual Pokémon in a team are separated by ].

4. Accessing the Pokédex (Dex)
The Dex module provides the entire Showdown database. By default, Dex is configured for the latest generation (Gen 9).

Basic Queries
javascript

const { Dex, toID } = require('@pkmn/sim');
// 1. Look up Pokémon Species
const pikachu = Dex.species.get('pikachu'); // Case/space-insensitive
console.log(pikachu.name);           // "Pikachu"
console.log(pikachu.num);            // 25
console.log(pikachu.types);          // ["Electric"]
console.log(pikachu.baseStats);      // { hp: 35, atk: 55, def: 40, spa: 50, spd: 50, spe: 90 }
console.log(pikachu.abilities);      // { '0': 'Static', 'H': 'Lightning Rod' }
console.log(pikachu.weightkg);       // 6
console.log(pikachu.tier);           // "LC"
console.log(pikachu.prevo);          // "Pichu"
console.log(pikachu.evos);           // ["Raichu", "Raichu-Alola"]
console.log(pikachu.exists);         // true (always check this for valid entries)
// Formes
const rotomWash = Dex.species.get('Rotom-Wash');
console.log(rotomWash.types);        // ["Electric", "Water"]
console.log(rotomWash.baseSpecies);  // "Rotom"
console.log(rotomWash.forme);        // "Wash"
// 2. Look up Abilities
const intimidate = Dex.abilities.get('Intimidate');
console.log(intimidate.name);        // "Intimidate"
console.log(intimidate.desc);        // Full description
console.log(intimidate.shortDesc);   // Brief in-battle description
console.log(intimidate.rating);      // Competitive viability rating (e.g. 3.5)
// 3. Look up Moves
const thunderbolt = Dex.moves.get('Thunderbolt');
console.log(thunderbolt.basePower);  // 90
console.log(thunderbolt.type);       // "Electric"
console.log(thunderbolt.category);   // "Special" ("Physical" | "Special" | "Status")
console.log(thunderbolt.accuracy);   // 100 (or true for bypass accuracy)
console.log(thunderbolt.pp);         // 15
console.log(thunderbolt.priority);   // 0
console.log(thunderbolt.target);     // "normal" (single target)
// 4. Look up Items
const leftovers = Dex.items.get('Leftovers');
console.log(leftovers.name);         // "Leftovers"
console.log(leftovers.desc);
// 5. Look up Natures & Types
const timid = Dex.natures.get('Timid');
console.log(timid.plus);             // 'spe'
console.log(timid.minus);            // 'atk'
const electric = Dex.types.get('Electric');
console.log(electric.damageTaken);   // Type matchup effectiveness table
Looking Up Learnsets
Learnset data tracks what moves a Pokémon can learn across generations and sources (level-up, TM, egg, tutor).

javascript

// Get raw learnset data
const learnsetData = Dex.species.getLearnsetData(pikachu.id);
console.log(learnsetData.learnset);
// Maps move ID to learning sources:
// {
//   thunderbolt: ['9M', '8M', '7M', ...], // '9M' = Gen 9 TM
//   volttackle: ['9E', '8E', ...],        // '9E' = Egg move
//   thundershock: ['9L1', '8L1', ...]     // '9L1' = Gen 9 Level 1
// }
// Get full array of all moves the species can legally learn (including pre-evolutions):
const allMoves = Dex.species.getFullLearnset(pikachu.id);
console.log(allMoves.includes('thunderbolt')); // true
Generation-Specific Data (Dex.mod / Dex.forGen)
javascript

// Access Gen 1 rules and mechanics:
const gen1Dex = Dex.mod('gen1');
const gen1Tackle = gen1Dex.moves.get('Tackle');
console.log(gen1Tackle.basePower); // 35 (was 35 in Gen 1, 40 in Gen 9)
const gen8Dex = Dex.forGen(8);
const zacian = gen8Dex.species.get('Zacian-Crowned');
console.log(zacian.baseStats.atk); // 170 (nerfed to 150 in Gen 9)
5. The Battle Protocol Explained
All messages sent from the simulator are newline-separated, pipe-delimited strings: |command|arg1|arg2|....

1. |turn|TURN_NUMBER
Signals that a new turn has begun.

text

|turn|1
2. |switch|POKEMON|DETAILS|HP STATUS
Signals a Pokémon entering the field.

text

|switch|p1a: Dragapult|Dragapult, L100, M|280/280
|switch|p2a: Ting-Lu|Ting-Lu, L100|450/450
POKEMON: Slot + nickname: p1a: Dragapult (in singles p1a, in doubles p1a / p1b).
DETAILS: Species, Level, Gender, shiny (e.g. Pikachu, L50, M, shiny).
HP STATUS: CurrentHP/MaxHP (and status if any, e.g. 240/300 brn).
Note: In omniscient / spectator streams, HP is often percentage: 100/100. In player streams (streams.p1), your own Pokémon shows exact HP (280/280), while opponent shows percentage (100/100).
Related variants: |drag| (forced switch via Roar / Whirlwind / Dragon Tail), |replace| (Illusion Zoroark reveal).
3. |move|POKEMON|MOVE|TARGET|[flags]
Signals a Pokémon executing an attack.

text

|move|p1a: Dragapult|Shadow Ball|p2a: Ting-Lu
|move|p2a: Cinderace|Pyro Ball|p1a: Corviknight|[miss]
|move|p1a: Corviknight|Roost|p1a: Corviknight
4. |-damage|POKEMON|HP STATUS|[from]SOURCE
Signals HP reduction (minor action, prefixed with -).

text

|-damage|p2a: Ting-Lu|380/450
|-damage|p1a: Dragapult|252/280|[from] item: Life Orb
|-damage|p1a: Dragapult|0 fnt
|faint|p1a: Dragapult
When HP reaches 0 fnt, the simulator emits |faint|POKEMON.

5. |request|JSON_STRING
This is the most critical message for a bot. It is sent to a specific player's stream when the simulator requires input.

Example parsed JSON structure:

json

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
        },
        {
          "move": "Draco Meteor",
          "id": "dracometeor",
          "pp": 8,
          "maxpp": 8,
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
      },
      {
        "ident": "p1: Kingambit",
        "details": "Kingambit, L100, M",
        "condition": "404/404",
        "active": false,
        "stats": { "atk": 405, "def": 276, "spa": 140, "spd": 206, "spe": 147 },
        "moves": ["kowtowcleave", "suckerpunch", "ironhead", "swordsdance"],
        "ability": "supremeoverlord",
        "item": "leftovers",
        "teraType": "Flying"
      }
    ]
  }
}
Request Scenarios to Handle in Bot Logic:
request.wait === true: The opponent is making a decision, or a simultaneous switch is pending. Do NOT write any choice to the stream.
request.teamPreview === true: Sent at battle start. Must send team 123456 or default.
request.forceSwitch: An array of booleans (e.g. [true]). Your active Pokémon fainted or was forced out. You must switch (switch <index>), moves are not allowed.
request.active: Standard turn. You can choose a move (move <slot>) or switch (switch <slot>).
6. Using BattlePlayer and RandomPlayerAI
How BattlePlayer Works Internally
BattlePlayer is the abstract base class in sim/battle-stream.ts. It provides an event-driven loop that:

Reads string chunks from this.playerStream.
Splits them into lines.
Detects |request| lines, parses the JSON payload, and calls this.receiveRequest(request).
Detects |error| lines and calls this.receiveError(error).
Provides this.choose(choiceString) to write actions back to this.playerStream.
Using Built-in RandomPlayerAI
javascript

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
Writing a Custom AI by Extending BattlePlayer
Here is a complete, working template for creating custom bots:

javascript

const { BattlePlayer } = require('@pkmn/sim');
class SmartBattleBot extends BattlePlayer {
  constructor(playerStream, debug = false) {
    super(playerStream, debug);
  }
  /**
   * Called automatically whenever the simulator issues a decision request.
   */
  receiveRequest(request) {
    // 1. If wait flag is true, simulator is waiting for opponent
    if (request.wait) {
      return;
    }
    // 2. Team Preview
    if (request.teamPreview) {
      // Pick default order (or send "team 123456" for custom order)
      this.choose('default');
      return;
    }
    // 3. Forced Switch (e.g. your active Pokémon just fainted)
    if (request.forceSwitch) {
      const validSwitchSlots = [];
      request.side.pokemon.forEach((p, index) => {
        // slot index in showdown is 1-indexed (index + 1)
        if (!p.active && !p.condition.endsWith('fnt')) {
          validSwitchSlots.push(index + 1);
        }
      });
      if (validSwitchSlots.length > 0) {
        // Pick the first available alive bench Pokémon
        this.choose(`switch ${validSwitchSlots[0]}`);
      } else {
        this.choose('default');
      }
      return;
    }
    // 4. Standard Turn: Active Move Selection
    if (request.active) {
      const activePokemon = request.active[0];
      const availableMoves = [];
      activePokemon.moves.forEach((m, index) => {
        // Only select moves that have PP and are not disabled
        if (!m.disabled && m.pp > 0) {
          availableMoves.push(index + 1); // 1-indexed
        }
      });
      if (availableMoves.length > 0) {
        // Option to terastallize if available:
        const teraSuffix = activePokemon.canTerastallize ? ' terastallize' : '';
        
        // Pick move 1 (or apply heuristic)
        const chosenMove = availableMoves[0];
        this.choose(`move ${chosenMove}${teraSuffix}`);
      } else {
        // Struggle or default fallback
        this.choose('move 1');
      }
      return;
    }
    // Fallback
    this.choose('default');
  }
  receiveError(error) {
    console.error(`[${this.constructor.name}] Battle Error:`, error.message);
  }
}
module.exports = { SmartBattleBot };
7. Available Battle Format Strings
Format IDs in Pokémon Showdown are normalized via toID() (all lowercase, alphanumeric only).

Common Gen 9 Formats
Standard Singles:
gen9ou – OverUsed (Smogon flagship)
gen9ubers – Ubers
gen9uu – UnderUsed
gen9ru – RarelyUsed
gen9nu – NeverUsed
gen9pu – PU
gen9lc – Little Cup (Level 5 unevolved)
gen9monotype – Monotype (all Pokémon share a type)
gen9anythinggoes – Anything Goes (no banlist, duplicate species/items allowed)
gen91v1 – 1v1 Singles
gen9customgame – Custom Game (No team restrictions, no level limits, no banned moves/abilities)
Random Battles (Teams generated automatically by simulator):
gen9randombattle – Standard competitive random battle
gen9randomdoublesbattle – Doubles random battle
gen9monotyperandombattle – Monotype random battle
gen9hackmonscup – Random battle with randomized moves/abilities
Doubles & VGC:
gen9doublesou – Doubles OverUsed
gen9doublesubers – Doubles Ubers
gen9vgc2024regh / gen9vgc2025... – Official Pokémon Video Game Championships rules
gen9doublescustomgame – Doubles Custom Game
Past Generations: Simply replace gen9 with gen8, gen7, gen6, gen5, gen4, gen3, gen2, or gen1:
gen8ou, gen7randombattle, gen5ou, gen1ou, etc.
Querying Formats Programmatically
javascript

const { Dex } = require('@pkmn/sim');
// Get all registered format objects:
const allFormats = Dex.formats.all();
allFormats.forEach(f => {
  if (f.id.startsWith('gen9')) {
    console.log(f.id, '-', f.name);
  }
});
// Inspect a specific format ruleset:
const format = Dex.formats.get('gen9ou');
console.log('Name:', format.name);         // "[Gen 9] OU"
console.log('Game Type:', format.gameType); // "singles"
console.log('Ruleset:', format.ruleset);   // ['Standard', 'Sleep Moves Clause', ...]
console.log('Banlist:', format.banlist);   // ['Uber', 'AG', 'Arena Trap', ...]
8. End-to-End Working Example: Full Battle Simulation
Here is a complete, standalone script demonstrating how to run a simulated match between a custom bot and a random player:

javascript

const { BattleStreams, RandomPlayerAI, Teams } = require('@pkmn/sim');
const { SmartBattleBot } = require('./smart-bot'); // Class defined in section 6
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
  // 4. Listen to battle events via the spectator/omniscient stream
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
  // For random battles, use Teams.generate:
  const p1Team = Teams.pack(Teams.generate('gen9randombattle'));
  const p2Team = Teams.pack(Teams.generate('gen9randombattle'));
  // 6. Launch the battle
  await streams.omniscient.write(`>start {"formatid":"gen9randombattle"}\n`);
  await streams.omniscient.write(`>player p1 {"name":"SmartBot","team":"${p1Team}"}\n`);
  await streams.omniscient.write(`>player p2 {"name":"RandomAI","team":"${p2Team}"}\n`);
}
runBattle().catch(console.error);
