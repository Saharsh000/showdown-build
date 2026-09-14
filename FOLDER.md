Listed directory pokeventure-image-main-2
Searched for files: *img*
Listed directory img
Ran command: `find src/img -maxdepth 2 -type d`
Ran command: `ls -la src`
Ran command: `ls -la "/Users/saharsh/Downloads/duh copy"`
Ran command: `ls -la src/event`
Ran command: `python3 -c "
import os

img_dir = 'src/img'
for root, dirs, files in os.walk(img_dir):
    rel = os.path.relpath(root, img_dir)
    non_ds = [f for f in files if f != '.DS_Store']
    print(f'{rel}: {len(non_ds)} files, {len(dirs)} dirs')
"`
Searched for "event"
Viewed index.php:1-150
Viewed pokedex.php:1-100
Viewed pokedex.php:101-260
Viewed pokedex.php:261-350
Viewed pokedex.php:351-426
Viewed index.php:151-312
Searched for "img/"
Ran command: `python3 -c "
import os, glob

# Check files in src/img directly
root_files = sorted(os.listdir('src/img'))
for f in root_files:
    p = os.path.join('src/img', f)
    if os.path.isfile(p):
        print(f'{f} ({os.path.getsize(p)} bytes)')
"`
Ran command: `python3 -c "
import os
print('--- bgs ---')
print(sorted(os.listdir('src/img/bgs')))
print('--- trainers ---')
print(sorted(os.listdir('src/img/trainers')))
"`
Ran command: `python3 -c "
import os

mini_files = [f for f in os.listdir('src/img/mini') if not f.startswith('.')]
shiny_files = [f for f in os.listdir('src/img/shiny') if not f.startswith('.') and f != 'female']
shiny_female = [f for f in os.listdir('src/img/shiny/female') if not f.startswith('.')]

print(f'mini: {len(mini_files)} files. Samples: {mini_files[:10]}')
print(f'shiny: {len(shiny_files)} files. Samples: {shiny_files[:10]}')
print(f'shiny/female: {len(shiny_female)} files. Samples: {shiny_female[:10]}')

# Check extensions
from collections import Counter
print('mini exts:', Counter(os.path.splitext(f)[1] for f in mini_files))
print('shiny exts:', Counter(os.path.splitext(f)[1] for f in shiny_files))
print('shiny/female exts:', Counter(os.path.splitext(f)[1] for f in shiny_female))

# Check naming patterns (numeric vs alphabetical)
mini_numeric = [f for f in mini_files if f.split('.')[0].isdigit()]
mini_non_numeric = [f for f in mini_files if not f.split('.')[0].isdigit()]
print(f'mini numeric: {len(mini_numeric)}, non-numeric: {len(mini_non_numeric)}')
if mini_non_numeric:
    print('mini non-numeric samples/all:', mini_non_numeric[:20])

shiny_numeric = [f for f in shiny_files if f.split('.')[0].isdigit()]
shiny_non_numeric = [f for f in shiny_files if not f.split('.')[0].isdigit()]
print(f'shiny numeric: {len(shiny_numeric)}, non-numeric: {len(shiny_non_numeric)}')
if shiny_non_numeric:
    print('shiny non-numeric samples/all:', shiny_non_numeric[:20])
"`
Ran command: `python3 -c "
import os
from collections import Counter

for dir_name in ['front', 'front-shiny', 'back', 'back-shiny']:
    path = os.path.join('src/img', dir_name)
    files = [f for f in os.listdir(path) if not f.startswith('.')]
    exts = Counter(os.path.splitext(f)[1] for f in files)
    print(f'{dir_name}: {len(files)} files. Extensions: {dict(exts)}')
"`
Ran command: `python3 -c "
import os

dirs = ['front', 'front-shiny', 'back', 'back-shiny']
all_names = {}
for d in dirs:
    all_names[d] = set(os.path.splitext(f)[0] for f in os.listdir(os.path.join('src/img', d)) if not f.startswith('.'))

# Let's see some non-standard pokemon names (e.g. contain special keywords or hyphens or events)
event_keywords = ['xmas', 'christmas', 'halloween', 'val', 'fest', 'event', 'summer', 'winter', 'autumn', 'fall', 'spring', 'easter', 'holiday', 'party', 'birthday', 'costume', 'hat', 'santa', 'spooky', 'shadow', 'dark', 'light', 'crystal', 'gold', 'silver', 'cyber', 'mecha', 'robot', 'ghost', 'angel', 'devil', 'demon', 'clover', 'lucky', 'witch', 'pumpkin', 'flower', 'bloom', 'star', 'sun', 'moon', 'aurora', 'cherry', 'sakura', 'rainbow', 'golden', 'frozen', 'firework', 'circus', 'chef', 'pirate', 'ninja', 'detective', 'pop', 'rock', 'belle', 'phd', 'libre', 'cosplay', 'cap', 'totem', 'eternal']

for d in dirs:
    print(f'=== Keywords matches in {d} ===')
    matched = []
    for name in sorted(all_names[d]):
        for kw in event_keywords:
            if kw in name:
                matched.append(name)
                break
    print(f'Total matches: {len(matched)}')
    print('Matches:', matched[:40])
"`
Ran command: `python3 -c "
import os
from collections import defaultdict

dirs = ['front', 'front-shiny', 'back', 'back-shiny']
all_files = set()
for d in dirs:
    for f in os.listdir(os.path.join('src/img', d)):
        if not f.startswith('.'):
            all_files.add(os.path.splitext(f)[0])

suffixes = defaultdict(list)
for name in all_files:
    if '-' in name:
        parts = name.split('-', 1)
        pokemon = parts[0]
        suffix = parts[1]
        suffixes[suffix].append(pokemon)

print('Unique suffixes count:', len(suffixes))
# Print sorted suffixes by how many pokemon have them or interesting suffixes
sorted_suffixes = sorted(suffixes.items(), key=lambda x: len(x[1]), reverse=True)
for s, p_list in sorted_suffixes[:50]:
    print(f'-{s}: {len(p_list)} pokemon (e.g. {p_list[:5]})')
"`
Ran command: `python3 -c "
import os
from collections import defaultdict

dirs = ['front', 'front-shiny', 'back', 'back-shiny']
all_files = set()
for d in dirs:
    for f in os.listdir(os.path.join('src/img', d)):
        if not f.startswith('.'):
            all_files.add(os.path.splitext(f)[0])

event_like_suffixes = defaultdict(set)
for name in all_files:
    for tag in ['halloween', 'xmas', 'bday', 'easter', 'vday', 'val', 'fest', 'party', 'spring', 'summer', 'autumn', 'winter', 'event', 'newyear', 'anniv', 'costume', 'hat', 'santa', 'spooky', 'bloom']:
        if tag in name:
            event_like_suffixes[tag].add(name)

for k, names in sorted(event_like_suffixes.items()):
    print(f'=== TAG: {k} ({len(names)} sprites) ===')
    print(sorted(list(names)))
"`
Ran command: `python3 -c "
import os

dirs = ['front', 'front-shiny', 'back', 'back-shiny']
all_bases = set()
all_full = set()
for d in dirs:
    for f in os.listdir(os.path.join('src/img', d)):
        if not f.startswith('.'):
            base = os.path.splitext(f)[0]
            all_full.add(base)
            all_bases.add(base.split('-')[0])

print(f'Total unique full sprite filenames across front/back: {len(all_full)}')
print(f'Total unique base pokemon names: {len(all_bases)}')

# Let's see what bases have '-' or unusual characters or custom names
# Check for any CAP (Create-A-Pokemon) or custom mons or Digimon or events
print('Sample all_bases:', sorted(list(all_bases))[:50])
"`
Ran command: `python3 -c "
import os

all_full = set()
for d in ['front', 'front-shiny', 'back', 'back-shiny']:
    for f in os.listdir(os.path.join('src/img', d)):
        if not f.startswith('.'):
            all_full.add(os.path.splitext(f)[0])

digimon = [x for x in all_full if x.endswith('mon') and not any(p in x for p in ['ditto', 'aipom', 'ambipom', 'castform'])]
# Filter actual digimon
print(f'Possible digimon ({len(digimon)}): {digimon[:30]}')
"`
Ran command: `ls src/img/front | head -n 10`
Created analyze.py
Ran command: `python3 /Users/saharsh/.gemini/antigravity/brain/349683a2-267b-4508-b1a3-1238624b5767/scratch/analyze.py`
Created analyze.py
Ran command: `python3 /Users/saharsh/.gemini/antigravity/brain/349683a2-267b-4508-b1a3-1238624b5767/scratch/analyze.py`
Ran command: `python3 -c "
import os
from PIL import Image

p = 'src/event/eternalbloom.png'
if os.path.exists(p):
    im = Image.open(p)
    print(f'eternalbloom.png: format={im.format}, size={im.size}, mode={im.mode}')
"`
Ran command: `file src/event/eternalbloom.png && sips -g pixelWidth -g pixelHeight src/event/eternalbloom.png`
Searched for "eternalbloom"
Ran command: `python3 -c "
import subprocess

res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = [f for f in res.stdout.splitlines() if f]
megas = [f for f in front if 'mega' in f.lower()]
print('Total megas in front:', len(megas))
print(sorted(megas))
"`
Ran command: `python3 -c "
import subprocess

res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = [f for f in res.stdout.splitlines() if f]

for region in ['alola', 'galar', 'hisui', 'paldea']:
    regs = [f for f in front if region in f.lower() and not any(e in f.lower() for e in ['halloween', 'xmas', 'bday'])]
    print(f'{region.capitalize()} ({len(regs)}):', sorted(regs))
"`
Ran command: `python3 -c "
import subprocess

res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = [f for f in res.stdout.splitlines() if f]
print('alola in front:', [f for f in front if 'alola' in f.lower()][:20])
print('galar in front:', [f for f in front if 'galar' in f.lower()][:20])
print('hisui in front:', [f for f in front if 'hisui' in f.lower()][:20])
print('paldea in front:', [f for f in front if 'paldea' in f.lower()][:20])
"`
Ran command: `python3 -c "
import subprocess

for d in ['front', 'front-shiny', 'back', 'back-shiny', 'mini', 'shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    files = res.stdout.splitlines()
    alolas = [f for f in files if 'alola' in f.lower()]
    print(f'{d}: {len(alolas)} alola files. Samples: {alolas[:5]}')
"`
Ran command: `python3 -c "
import subprocess

for d in ['front', 'front-shiny', 'back', 'back-shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    files = res.stdout.splitlines()
    for reg in ['alola', 'galar', 'hisui', 'paldea']:
        matches = [f for f in files if reg in f.lower() and 'halloween' not in f.lower()]
        print(f'{d} {reg}: {len(matches)}')
    break # Just front
"`
Ran command: `python3 -c "
import subprocess

res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
files = res.stdout.splitlines()
for reg in ['alola', 'galar', 'hisui', 'paldea']:
    matches = sorted([f for f in files if reg in f.lower() and 'halloween' not in f.lower()])
    print(f'=== {reg.upper()} ({len(matches)}) ===\n' + ', '.join(matches))
"`
Ran command: `python3 -c "
import subprocess

cap_names = [
    'syclant', 'revenankh', 'pyroak', 'fidgit', 'stratagem', 'arghonaut', 'kitsunoh', 'cyclohm',
    'colossoil', 'krilowatt', 'voodoom', 'tomohawk', 'necturna', 'mollux', 'aurumoth', 'malaconda',
    'cawmodore', 'volkraken', 'plasmanta', 'naviathan', 'crucibelle', 'kerfluffle', 'pajantom',
    'jumbao', 'caribolt', 'smokomodo', 'snaelstrom', 'equilibra', 'astrolotl', 'miasmaw',
    'syclar', 'embirch', 'flarelm', 'breezi', 'scratchet', 'necturine', 'cupra', 'argalis',
    'brattler', 'cawdet', 'volkritter', 'snugglow', 'floatoy', 'caimanoe', 'pluffle', 'rebble',
    'tactite', 'privatyke', 'nohface', 'monohm', 'duohm', 'voodoll', 'mumbao', 'fawnifer',
    'electrelk', 'smogecko', 'smoguana', 'swirlpool', 'coribalis', 'justyke', 'solotl', 'miasmite'
]

res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = [f for f in res.stdout.splitlines() if f]
found_cap = [f for f in front if any(f.startswith(c) for c in cap_names)]
print(f'CAP sprites found in front ({len(found_cap)}):', sorted(found_cap))
"`
Ran command: `python3 -c "
import subprocess

with open('/Users/saharsh/.gemini/antigravity/brain/349683a2-267b-4508-b1a3-1238624b5767/scratch/analyze.py') as f:
    pass

# Check how many digimon are in each directory
for d in ['front', 'front-shiny', 'back', 'back-shiny', 'mini', 'shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    files = res.stdout.splitlines()
    mon_files = [f for f in files if f.endswith('mon.gif') or f.endswith('mon.png')]
    print(f'{d}: {len(mon_files)} ending with mon')
"`
Ran command: `python3 -c "
import subprocess

for d in ['mini', 'shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    files = res.stdout.splitlines()
    for kw in ['halloween', 'xmas', 'bday', 'easter', 'vday', 'event', 'bloom']:
        matches = [f for f in files if kw in f.lower()]
        if matches:
            print(f'{d} {kw}: {matches}')
"`
Ran command: `python3 -c "
import subprocess

for d in ['mini', 'shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    files = res.stdout.splitlines()
    nums = [int(f.split('.')[0]) for f in files if f.split('.')[0].isdigit()]
    print(f'{d}: min={min(nums)}, max={max(nums)}, total numeric={len(nums)}')
"`
Ran command: `python3 -c "
import os

files_to_check = [
    'src/img/front-shiny/Sword_Shield_Gym_Challenge_artwork.webp',
    'src/img/front-shiny/unown-u.gif.crdownload',
    'src/img/back-shiny/bb.jpg',
    'src/img/back-shiny/fkf.jpeg',
]
for p in files_to_check:
    if os.path.exists(p):
        print(f'{p}: {os.path.getsize(p)} bytes')

import glob
for f in glob.glob('src/img/back-shiny/*.html'):
    print(f'{f}: {os.path.getsize(f)} bytes')
    with open(f, 'rb') as fp:
        print('  start:', fp.read(100))
"`
Ran command: `python3 -c "
import subprocess

res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = res.stdout.splitlines()

halloween = sorted([f for f in front if 'halloween' in f.lower() or 'hallo' in f.lower()])
print(f'Total Halloween in front ({len(halloween)}):')
for h in halloween:
    print(' ', h)
"`
Ran command: `python3 -c "
import subprocess

for d in ['front', 'front-shiny', 'back', 'back-shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    files = res.stdout.splitlines()
    hallo = [f for f in files if 'halloween' in f.lower() or 'hallo' in f.lower()]
    print(f'{d}: {len(hallo)} files')
"`
Ran command: `python3 -c "
import subprocess

sets = {}
for d in ['front', 'front-shiny', 'back', 'back-shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    sets[d] = set(f for f in res.stdout.splitlines() if 'halloween' in f.lower() or 'hallo' in f.lower())

all_hallo = set.union(*sets.values())
for f in sorted(all_hallo):
    pres = [d for d in sets if f in sets[d]]
    if len(pres) < 4:
        print(f'{f}: present in {pres}')
"`
Ran command: `python3 -c "
import subprocess

sets = {}
for d in ['front', 'front-shiny', 'back', 'back-shiny']:
    res = subprocess.run(['ls', '-1', f'src/img/{d}'], capture_output=True, text=True)
    sets[d] = set(f for f in res.stdout.splitlines() if 'xmas' in f.lower() or 'christmas' in f.lower())

all_xmas = set.union(*sets.values())
print(f'Total unique xmas files: {len(all_xmas)}')
for f in sorted(all_xmas):
    pres = [d for d in sets if f in sets[d]]
    if len(pres) < 4:
        print(f'{f}: present in {pres}')
"`
Ran command: `ls -1 src/img/front | grep -i xmas | wc -l`
Ran command: `for d in front front-shiny back back-shiny; do echo -n "$d: "; ls -1 src/img/$d | grep -i xmas | wc -l; done`
Ran command: `for d in front front-shiny back back-shiny; do echo -n "$d: "; ls -1 src/img/$d | grep -i bday | wc -l; done`
Ran command: `for d in front front-shiny back back-shiny; do echo -n "$d: "; ls -1 src/img/$d | grep -i easter | wc -l; done`
Ran command: `for d in front front-shiny back back-shiny; do echo -n "$d: "; ls -1 src/img/$d | grep -i vday; done`
Ran command: `ls -1 src/img/front | grep -E -i 'deerling|sawsbuck'`
Ran command: `ls -1 src/img/front | grep -i pikachu`
Ran command: `ls -1 src/img/front | grep -i eevee`
Ran command: `python3 -c "
import subprocess

res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = res.stdout.splitlines()

for mon in ['arceus', 'silvally', 'vivillon', 'alcremie', 'unown', 'rotom', 'furfrou', 'castform', 'deoxys', 'minior']:
    matches = [f for f in front if f.startswith(mon)]
    print(f'{mon} ({len(matches)}): {matches[:5]}...')
"`
Ran command: `python3 -c "
import subprocess

gen9_starters = ['sprigatito', 'floragato', 'meowscarada', 'fuecoco', 'crocalor', 'skeledirge', 'quaxly', 'quaxwell', 'quaquaval']
res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = res.stdout.splitlines()

found_gen9 = [f for f in front if any(f.startswith(s) for s in gen9_starters)]
print('Gen 9 starters in front:', found_gen9)

# Let's search other gen 9 mons: lechonk, pawmi, tinkatink, koraidon, miraidon
other_gen9 = ['lechonk', 'pawmi', 'tinkatink', 'tinkaton', 'koraidon', 'miraidon', 'annihilape', 'kingambit', 'gholdengo', 'tatsugiri', 'dondozo', 'baxcalibur', 'iron', 'scream', 'flutter', 'roaring', 'great', 'brute', 'slither', 'sandy', 'ogerpon', 'terapagos']
found_other = [f for f in front if any(f.startswith(s) for s in other_gen9)]
print('Other Gen 9 mons in front:', found_other)
"`
Ran command: `python3 -c "
import subprocess

dlc = ['pecharunt', 'archaludon', 'dipplin', 'hydrapple', 'poltchageist', 'sinistcha', 'okidogi', 'munkidori', 'fezandipiti', 'ogerpon']
res = subprocess.run(['ls', '-1', 'src/img/front'], capture_output=True, text=True)
front = res.stdout.splitlines()

found_dlc = [f for f in front if any(f.startswith(s) for s in dlc)]
print('DLC mons in front:', found_dlc)
"`
Ran command: `ls -1 src/img/front | grep -E -i 'ogerpon|terapagos|pecharunt'`
Ran command: `du -sh src/img/* src/img src/event`

Here is a comprehensive analysis of the `img` directory in the **Pokeventure** project (`/src/img`), covering directory structure, system usage, assets, and an exhaustive breakdown of the **event sprites**.

---

### 1. High-Level Overview & Architecture

The `src/img/` folder (~**678 MB**, **11,048 files**) serves as the central graphics and sprite asset repository for the Pokeventure web rendering engine. It powers multiple endpoints in the codebase:
- **`index.php`**: Battle canvas renderer (merges background, UI, and animated Pokémon front/back battle GIFs).
- **`pokedex.php`**: Pokédex list renderer using `pokemonicons-sheet.png` and National Dex mapping.
- **`gym_show.php`**: Gym leader battle preview with trainer sprites and gym backgrounds.
- **`bingo.php`**, **`slot.php`**, **`vote.php`**: Mini-games and daily reward generators utilizing `mini/` and `shiny/` sprites, backgrounds, stamps, and ticket artwork.

---

### 2. Directory Structure & File Counts

| Directory / Path | File Count | Formats | Size | Primary Purpose |
| :--- | :---: | :--- | :---: | :--- |
| **`src/img/` (Root)** | 40 files | `.png`, `.jpg`, `.webp`, `.ttf`, `.otf`, `.gif` | ~1.5 MB | UI elements, battle frames, fonts, spritesheets, mini-game assets |
| **`front/`** | 2,183 files | `.gif` (2,141), `.png` (42) | 184 MB | Standard animated front battle sprites (opponent view) |
| **`front-shiny/`** | 2,154 files | `.gif` (2,111), `.png` (41), `.webp` (1), `.crdownload` (1) | 183 MB | Shiny animated front battle sprites |
| **`back/`** | 2,076 files | `.gif` (2,075), `.png` (1) | 147 MB | Animated back battle sprites (player view) |
| **`back-shiny/`** | 2,052 files | `.gif` (2,045), `.html` (5), `.jpeg` (1), `.jpg` (1) | 146 MB | Shiny animated back battle sprites |
| **`shiny/`** | 1,493 files | `.png` (1,393 root + 100 in `female/`) | 6.8 MB | Static shiny model icons indexed by PokéAPI / Showdown IDs |
| **`mini/`** | 1,142 files | `.png` (1,142) | 5.2 MB | Mini menu / box sprites (Dex IDs 0–898 + forme variants) |
| **`bgs/`** | 33 files | `.jpeg` (19), `.jpg` (14) | 2.7 MB | Battle field backgrounds (`-1` through `20`) |
| **`trainers/`** | 14 files | `.gif` (14) | 1.1 MB | Animated gym leader / boss sprites |
| **`src/event/`** | 1 file | `.png` | 5.6 MB | High-res promotional event banner (`eternalbloom.png`) |

---

### 3. Detailed Breakdown of Event Sprites

Pokeventure features custom-themed event Pokémon with custom animations and recolors. In total, there are **5 event themes** across **~200 distinct Pokémon event variants**:

#### A. Halloween Event (`-halloween`) — **117 Files (~110 Unique Pokémon/Forms)**
Halloween is the largest event roster, featuring spooky costumes, witch hats, pumpkin lanterns, ghostly color palettes, and skeletal/dark recolors.

* **Eeveelutions**: Eevee, Vaporeon, Jolteon, Flareon, Espeon, Umbreon, Leafeon, Glaceon, Sylveon.
* **Ghost & Dark Types**: Sableye, Duskull, Dusclops, Dusknoir, Spiritomb, Litwick, Lampent, Chandelure, Zorua, Zoroark, Phantump, Trevenant, Dreepy, Drakloak, Dragapult, Darkrai, Marshadow, Hoopa (`hoopa-halloween`, `hoopa-unbound-halloween`).
* **Dragon Lines**: Gible, Gabite, Garchomp; Deino, Zweilous, Hydreigon; Noibat, Noivern; Tyrunt, Tyrantrum.
* **Alolan Halloween Variants**:
  - `geodude-alola-halloween`, `gravaler-alola-halloween`, `golem-alola-halloween`
  - `grimer-alola-halloween`, `muk-alola-halloween`
  - `marowak-alola-halloween`
  - `vulpix-alola-halloween`, `ninetales-alola-halloween`
* **Castform Weather Forms**: Normal, `castform-rainy-halloween`, `castform-snowy-halloween`, `castform-sunny-halloween`.
* **Legendaries & Mythicals**: Darkrai, Marshadow, Victini, Hoopa, Zacian, Zamazenta.
* **Other Pokémon**: Aegislash, Amaura, Appletun, Applin, Araquanid, Aurorus, Bibarel, Bidoof, Bunnelby, Cinccino, Cubone, Dedenne, Dewpider, Diggersby, Ditto, Doublade, Flapple, Floette (Eternal), Girafarig, Gligar, Gliscor, Herdier, Hitmonchan, Hitmonlee, Honedge, Jigglypuff, Kecleon, Lapras, Larvitar, Pupitar, Tyranitar, Lillipup, Lombre, Lotad, Ludicolo, Machop, Machoke, Machamp, Meowstic, Minccino, Onix, Paras, Parasect, Pikachu, Raichu, Raticate, Rattata, Seviper, Shuckle, Slakoth, Vigoroth, Slaking, Steelix, Stoutland, Sudowoodo, Tentacool, Tentacruel, Togepi, Togetic, Togekiss, Wigglytuff, Wobbuffet.

#### B. Christmas / Holiday Event (`-xmas`) — **60 Unique Pokémon**
Winter and holiday-themed sprites featuring Santa hats, scarves, festive ornaments, gift ribbons, and snowy recolors. Available in all 4 battle directories (`front`, `front-shiny`, `back`, `back-shiny`):

* **Kanto Starters & Classics**: `bulbasaur-xmas`, `charmander-xmas`, `squirtle-xmas`, `pikachu-xmas`, `snorlax-xmas`, `vulpix-xmas`, `jigglypuff-xmas`, `oddish-xmas`, `psyduck-xmas`, `weepinbell-xmas`, `voltorb-xmas`, `porygon-xmas`, `lapras-xmas`, `ditto-xmas`.
* **Legendaries & Mythicals**: `celebi-xmas`, `jirachi-xmas`, `shaymin-xmas`, `meloetta-xmas`, `xerneas-xmas`, `zekrom-xmas`.
* **Ice & Winter Themed**: `snover-xmas`, `deerling-xmas`, `sawsbuck-xmas`, `eiscue-xmas`, `spheal-xmas`, `stantler-xmas`.
* **Gen 8 & 9 Holiday Additions**: `lechonk-xmas`, `tandemaus-xmas`, `wooloo-xmas`, `morpeko-xmas`, `milcery-xmas`, `hatenna-xmas`.
* **Other Festive Pokémon**: `bellossom-xmas`, `cherubi-xmas`, `chingling-xmas`, `comfey-xmas`, `drifloon-xmas`, `gliscor-xmas`, `kirlia-xmas`, `komala-xmas`, `lotad-xmas`, `mimejr-xmas`, `mimikyu-xmas`, `musharna-xmas`, `pineco-xmas`, `ponyta-xmas`, `riolu-xmas`, `rockruff-xmas`, `rotom-xmas`, `shinx-xmas`, `skiddo-xmas`, `solosis-xmas`, `sylveon-xmas`, `timburr-xmas`, `togepi-xmas`, `wormadam-xmas`, `wurmple-xmas`, `yamask-xmas`, `zigzagoon-xmas`.

#### C. Birthday Event (`-bday`) — **19 Legendary & Mythical Pokémon**
Exclusive golden / birthday-themed celebratory sprites for major legendaries from Gens 1–3:
- **Kanto (5)**: `articuno-bday`, `zapdos-bday`, `moltres-bday`, `mewtwo-bday`, `mew-bday`
- **Johto (5)**: `raikou-bday`, `entei-bday`, `suicune-bday`, `lugia-bday`, `celebi-bday`
- **Hoenn (9)**: `regirock-bday`, `regice-bday`, `registeel-bday`, `latias-bday`, `latios-bday`, `kyogre-bday`, `groudon-bday`, `rayquaza-bday`, `jirachi-bday`
> **Note**: Birthday sprites exist in `front/` and `back/`, but **do not** have shiny counterparts (`front-shiny` and `back-shiny` are not present).

#### D. Easter Event (`-easter`) — **3 Pokémon**
Spring and Easter-egg themed sprites:
- `bunnelby-easter` (Easter bunny)
- `eevee-easter` (Easter egg basket/ears)
- `exeggcute-easter` (Decorated painted eggs)
*(Available in all 4 battle directories: `front`, `front-shiny`, `back`, `back-shiny`)*

#### E. Valentine's Day Event (`-vday` / `-vdays`) — **2 Pokémon**
Heart-themed sprites for the Eon Duo:
- `latias-vday`, `latios-vday` (Normal Valentine's forms)
- `latias-vdays`, `latios-vdays` (Shiny Valentine's variants)

#### F. Standalone Event Graphics (`src/event/`)
- **`eternalbloom.png`**: High-resolution (2730 × 1536 px, 5.6 MB) promotional artwork / banner featuring AZ’s Floette (Eternal Flower form) in full bloom.

---

### 4. Other Notable & Custom Content in the Sprites

Beyond official Pokémon and seasonal events, the sprite folders include notable custom and crossover additions:

1. **Digimon Roster (241 Digimon)**:
   - There are **241 Digimon** complete with full battle sets in `front/`, `front-shiny/`, `back/`, and `back-shiny/` (e.g., Agumon, Greymon, WarGreymon, Omnimon, Beelzemon, Renamon, Lucemon/Daemon, Angemon, Angewomon, Guilmon, etc.).
2. **Custom / Fanmade Mega Evolutions & Mega-Z (40+ Custom Megas)**:
   - Alongside official Megas, Pokeventure includes custom Mega sprites:
     - `raichu-megax`, `raichu-megay`, `clefable-mega`, `dragonite-mega`, `starmie-mega`, `victreebel-mega`
     - `meganium-mega`, `feraligatr-mega`, `skarmory-mega`, `chimecho-mega`
     - `staraptor-mega`, `froslass-mega`, `darkrai-mega`
     - `emboar-mega`, `excadrill-mega`, `scolipede-mega`, `scrafty-mega`, `chandelure-mega`, `eelektross-mega`, `golurk-mega`
     - `chesnaught-mega`, `delphox-mega`, `greninja-mega`, `meowstic-f-mega`, `meowstic-m-mega`, `dragalge-mega`, `malamar-mega`, `hawlucha-mega`, `floette-mega`, `zygarde-mega`
     - `drampa-mega`, `golisopod-mega`, `magearna-mega`, `zeraora-mega`, `falinks-mega`
     - Gen 9 Megas: `baxcalibur-mega`, `glimmora-mega`, `scovillain-mega`, `tatsugiri-curly-mega`, `tatsugiri-droopy-mega`, `tatsugiri-stretchy-mega`
     - **Mega-Z Series**: `absol-megaz`, `garchomp-megaz`, `lucario-megaz`.
3. **Gigantamax (G-Max)**: 33 Gigantamax sprites (`charizard-gmax`, `gengar-gmax`, `snorlax-gmax`, `pikachu-gmax`, etc.).
4. **Regional Variants**:
   - **Alolan (21)**: Sandshrew, Vulpix, Raichu, Marowak, Exeggutor, Muk, Golem, etc.
   - **Galarian (20)**: Weezing, Ponyta, Rapidash, Corsola, Zigzagoon, Birds trio, Slowbro/king, etc.
   - **Hisuian (17)**: Zorua, Zoroark, Decidueye, Typhlosion, Samurott, Goodra, Arcanine, etc.
   - **Paldean (7)**: Tauros (Combat, Blaze, Aqua) and Wooper.
5. **Smogon CAP (Create-A-Pokemon)**: 37 species including Syclant, Revenankh, Pyroak, Kitsunoh, Tomohawk, Crucibelle, Astrolotl, etc.
6. **Forme Variations**:
   - **Alcremie**: 118 sprite combinations (every cream and sweet variation).
   - **Vivillon**: 20 regional wing patterns.
   - **Unown**: 28 forms (A–Z, ?, !).
   - **Minior**: 8 core colors + meteor form.
   - **Furfrou**: 10 trims.

---

### 5. Root Assets & Supporting Graphics

| Filename | Dimensions / Type | Usage |
| :--- | :--- | :--- |
| `0.png`, `1.png` | PNG | Battle battlebase platforms / shadows |
| `ui.png`, `ui-player.png`, `ui-enemy.png` | PNG | HP bars and battle HUD overlays |
| `ballfull.png`, `balldead.png`, `ball.png` | PNG | Party status Pokéball icons |
| `male.png`, `female.png` | PNG | Gender indicators |
| `pokemonicons-sheet.png` | 321 KB PNG | Full icon spritesheet used by `pokedex.php` |
| `itemicons-sheet.png` | 76 KB PNG | Item spritesheet used by `vote_template.php` |
| `pokemon_fire_red.ttf`, `pokemon-font.ttf`, `Whitney-Book.ttf`, `whitneybook.otf` | TTF / OTF | Fonts for rendering player names, levels, and Pokédex text |
| `bg_vote.png`, `streak.png`, `candy.png`, `egg.png`, `pass.png`, `stamp.png` | PNG / JPG | Assets for daily vote streak rewards (`vote.php`) |
| `bingobg.jpg`, `logo.png` | JPG / PNG | 5x5 daily Pokémon bingo board (`bingo.php`) |
| `slot.jpg`, `shinyticket.jpg`, `shinyticket.png` | JPG / PNG | Casino slot machine mini-game (`slot.php`) |
| `missingno.png` | PNG | Fallback image when a requested Pokémon is not found |
| `weakness.png` | PNG | Type effectiveness visual matrix |
| `bgs/` (`-1.jpeg` through `20.jpeg`) | 33 files | Diverse battle landscapes (gym, grass, mountain, sea, cave, space) |
| `trainers/` (14 GIFs) | 14 files | Kanto gym leaders and elite trainers (Brock, Misty, Sabrina, Giovanni, etc.) |

---

### 6. Notable File Inconsistencies & Quirks Discovered

A review of the sprite folders revealed several naming and scraping artifacts:
1. **Scraped HTML Directory Indexes in `back-shiny/`**: Five files named `te╠üle╠üchargement (1).html` through `(4).html` and `te╠üle╠üchargement.html` (~279 KB each) are raw Apache directory listings (`Index of /sprites/ani-back-shiny`) downloaded during asset scraping.
2. **Incomplete Download**: `src/img/front-shiny/unown-u.gif.crdownload` is an unfinished Google Chrome download.
3. **Spaces in Filenames in `front/`**:
   - `paras hallo normal.gif` and `parasect hallo normal.gif` (the other folders use `paras-halloween.gif` and `parasect-halloween.gif`).
   - `ditto-halloween copy.gif` (duplicate of `ditto-halloween.gif`).
   - `magcargo (1).gif` (in `back/`) and `unown-l (1).gif` (in `back-shiny/`).
4. **Typos in Filenames**:
   - `hydriegon-halloween.gif` (spelled `ie` instead of `ei`) in `front/` and `front-shiny/`, while `back/` uses `hydreigon-halloween.gif` (`back-shiny/` contains both).
   - `lilipup-halloween.gif` (single `l`) in `front/`, while other folders use `lillipup-halloween.gif`.
   - `gravaler-alola-halloween.gif` (spelled `gravaler` instead of `graveler`).
