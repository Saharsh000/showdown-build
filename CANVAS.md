# Pokemon Battle Scene Renderer with node-canvas

A complete technical guide for building Pokemon battle scene renders in Node.js using `node-canvas`, with Discord.js integration.

---

## Table of Contents

1. [Canvas Basics](#1-canvas-basics)
2. [Loading Images](#2-loading-images)
3. [Custom Fonts](#3-custom-fonts)
4. [HP Bars](#4-hp-bars)
5. [Sprite Compositing](#5-sprite-compositing)
6. [Discord.js Export](#6-discord-export)
7. [Transparency & Alpha](#7-transparency--alpha)
8. [GIF Gotchas](#8-gif-gotchas)
9. [Complete Example](#9-complete-example)
10. [Alternatives](#10-alternatives)

---

## 1. Canvas Basics

node-canvas replicates the HTML5 Canvas API using Cairo graphics engine.

### Creating a Canvas

```javascript
const { createCanvas } = require('canvas');

// Standard Pokemon Showdown dimensions
const width = 518;
const height = 288;
const canvas = createCanvas(width, height);
const ctx = canvas.getContext('2d');

// Enable crisp rendering for pixel art
ctx.patternQuality = 'nearest';
ctx.quality = 'nearest';

// Fill background
ctx.fillStyle = '#202020';
ctx.fillRect(0, 0, width, height);
```

---

## 2. Loading Images

### Supported Input Sources

`loadImage()` accepts:
- Local file paths
- Remote URLs
- Data URIs
- Buffer instances

All return a **Promise** resolving to an Image object.

### Examples

```javascript
const { loadImage } = require('canvas');
const axios = require('axios');

// Local file
const bg = await loadImage('./assets/bgs/forest.jpeg');

// Remote URL via Buffer (recommended)
const response = await axios.get(
  'https://play.pokemonshowdown.com/sprites/gen5/charizard.png',
  { responseType: 'arraybuffer', timeout: 5000 }
);
const sprite = await loadImage(Buffer.from(response.data));
```

### GIF Support

- ✅ Supported via giflib
- ⚠️ **Only renders Frame 0** — no animation
- ⚠️ Use static PNGs instead (see [GIF Gotchas](#8-gif-gotchas))

---

## 3. Custom Fonts

### Register TTF/OTF Fonts

```javascript
const { registerFont, createCanvas } = require('canvas');
const path = require('path');

// Register BEFORE drawing
registerFont(path.resolve(__dirname, './img/pokemon_fire_red.ttf'), {
  family: 'Pokemon Fire Red'
});

// Use in context
const canvas = createCanvas(518, 288);
const ctx = canvas.getContext('2d');
ctx.font = '18px "Pokemon Fire Red"';
ctx.fillStyle = '#000000';
ctx.textBaseline = 'top';
ctx.fillText('PIKACHU', 31, 28);
```

### Drop Shadow Effect

```javascript
const text = 'PIKACHU';
const x = 31, y = 28;

// Shadow
ctx.fillStyle = '#686868';
ctx.fillText(text, x + 1, y + 1);

// Main text
ctx.fillStyle = '#202020';
ctx.fillText(text, x, y);
```

### Text Measurement (Right-Align)

```javascript
const hpText = '120/150';
const textWidth = ctx.measureText(hpText).width;
ctx.fillText(hpText, 470 - textWidth, 226);
```

### Important Notes

- ⚠️ Call `registerFont()` **before** `createCanvas()` or setting `ctx.font`
- ⚠️ Always use **absolute paths** with `path.resolve()`
- ⚠️ Wrap font family in quotes: `ctx.font = '18px "My Font"'`

---

## 4. HP Bars

### Structure

1. **Background track** — dark gray/black
2. **Filled bar** — proportional to current HP
3. **Dynamic color** — Green (>50%), Yellow (20–50%), Red (<20%)

### Implementation

```javascript
function drawHpBar(ctx, currentHp, maxHp, x, y, maxWidth = 95, height = 5) {
  const hpRatio = Math.max(0, Math.min(1, currentHp / maxHp));
  const currentWidth = Math.round(hpRatio * maxWidth);

  // Background track
  ctx.fillStyle = '#404040';
  ctx.fillRect(x, y, maxWidth, height);

  // Determine color
  let barColor = '#16bf1d'; // Green (healthy)
  if (hpRatio <= 0.20) {
    barColor = '#ee4646';   // Red (critical)
  } else if (hpRatio <= 0.50) {
    barColor = '#f8bb34';   // Yellow (damaged)
  }

  // Draw filled bar
  if (currentWidth > 0) {
    ctx.fillStyle = barColor;
    ctx.fillRect(x, y, currentWidth, height);
  }
}

// Usage (for 518x288 canvas)
drawHpBar(ctx, enemyCurrentHp, enemyMaxHp, 99, 53, 95, 5);    // Enemy
drawHpBar(ctx, playerCurrentHp, playerMaxHp, 376, 218, 95, 5); // Player
```

---

## 5. Sprite Compositing

### Layer Order (Bottom to Top)

1. Background image
2. Enemy Pokemon sprite (top-right)
3. Player Pokemon sprite (bottom-left)
4. UI overlay/HUD
5. HP bars
6. Gender icons & status ailments
7. Text (names, levels, HP numbers)

### Positioning Formulas

```javascript
// Enemy Sprite (Top-Right area)
const enemyX = 518 - 30 - 105 - (enemySprite.width / 2);
const enemyY = Math.max(0, 32 + 80 + 32 - enemySprite.height);
ctx.drawImage(enemySprite, enemyX, enemyY);

// Player Sprite (Bottom-Left area)
const playerX = 50 + 23 + 64 - (playerSprite.width / 2);
const playerY = 288 - 33 - playerSprite.height;
ctx.drawImage(playerSprite, playerX, playerY);
```

### Flip Sprite Horizontally

```javascript
function drawFlippedSprite(ctx, image, x, y) {
  ctx.save();
  ctx.translate(x + image.width, y);
  ctx.scale(-1, 1);
  ctx.drawImage(image, 0, 0);
  ctx.restore();
}
```

---

## 6. Discord Export

### Synchronous Buffer Export

```javascript
const { AttachmentBuilder, EmbedBuilder } = require('discord.js');

// Export as PNG buffer
const buffer = canvas.toBuffer('image/png');

// Wrap in AttachmentBuilder
const attachment = new AttachmentBuilder(buffer, { name: 'battle.png' });
```

### Option A: Direct Attachment

```javascript
await interaction.reply({
  content: 'A wild battle started!',
  files: [attachment]
});
```

### Option B: Attachment in Embed

```javascript
const embed = new EmbedBuilder()
  .setTitle('Wild Pokémon Encounter!')
  .setColor(0x5865F2)
  .setImage('attachment://battle.png');

await interaction.reply({
  embeds: [embed],
  files: [attachment]
});
```

---

## 7. Transparency & Alpha

### Default Blending

- node-canvas defaults to `globalCompositeOperation = 'source-over'`
- Transparent PNGs blend correctly with background

### Canvas Initialization

- Canvas starts **transparent** by default: `rgba(0, 0, 0, 0)`
- ⚠️ Do NOT use black fills for initialization

### Pixel Crispness

node-canvas uses different properties than browsers:

```javascript
// For crisp pixel art (nearest-neighbor scaling)
ctx.patternQuality = 'nearest';
ctx.quality = 'nearest';

// (Browser equivalent: ctx.imageSmoothingEnabled = false)
```

---

## 8. GIF Gotchas

### Limitations

- ❌ Only Frame 0 is rendered
- ❌ **No animation support** — static image only
- ❌ Inter-frame delta compression may corrupt sprites
- ❌ Transparent palette indices may become solid black

### Recommended Solution

Use **Pokemon Showdown's static PNG repositories**:

```
Front Sprites:      https://play.pokemonshowdown.com/sprites/gen5/{name}.png
Shiny Front:        https://play.pokemonshowdown.com/sprites/gen5-shiny/{name}.png
Back Sprites:       https://play.pokemonshowdown.com/sprites/gen5-back/{name}.png
Shiny Back:         https://play.pokemonshowdown.com/sprites/gen5-back-shiny/{name}.png
PokeAPI Fallback:   https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/{id}.png
```

### Network Timeouts

- ⚠️ Direct `loadImage('https://...')` lacks timeouts
- ✅ Use `axios` with timeout and retry logic

---

## 9. Complete Example

### Full Battle Scene Renderer

```javascript
const { createCanvas, loadImage, registerFont } = require('canvas');
const axios = require('axios');
const path = require('path');

// Register fonts once at module load
registerFont(path.resolve(__dirname, './img/pokemon_fire_red.ttf'), {
  family: 'Pokemon Fire Red'
});

async function fetchImageBuffer(url) {
  const res = await axios.get(url, {
    responseType: 'arraybuffer',
    timeout: 5000
  });
  return Buffer.from(res.data);
}

/**
 * Renders a Pokemon battle scene and returns a PNG buffer.
 * @param {Object} config - Battle configuration
 * @returns {Promise<Buffer>} PNG image buffer
 */
async function renderBattleScene({
  bgLocation = 'town',
  p1 = {
    name: 'Charizard',
    level: 50,
    currentHp: 150,
    maxHp: 150,
    gender: 'M',
    sprite: 'charizard',
    isShiny: false
  },
  p2 = {
    name: 'Pikachu',
    level: 48,
    currentHp: 80,
    maxHp: 100,
    gender: 'F',
    sprite: 'pikachu',
    isShiny: false
  }
}) {
  const width = 518;
  const height = 288;
  const canvas = createCanvas(width, height);
  const ctx = canvas.getContext('2d');

  // Crisp pixel art scaling
  ctx.patternQuality = 'nearest';

  // Load static assets in parallel
  const [bgImg, uiImg, maleIcon, femaleIcon] = await Promise.all([
    loadImage(path.resolve(__dirname, `./img/bgs/${bgLocation}.jpeg`))
      .catch(() => loadImage(path.resolve(__dirname, './img/bgs/-1.jpeg'))),
    loadImage(path.resolve(__dirname, './img/ui.png')),
    loadImage(path.resolve(__dirname, './img/male.png')),
    loadImage(path.resolve(__dirname, './img/female.png'))
  ]);

  // Load Pokemon sprites from Showdown CDN
  const enemySpriteUrl = `https://play.pokemonshowdown.com/sprites/${
    p2.isShiny ? 'gen5-shiny' : 'gen5'
  }/${p2.sprite.toLowerCase()}.png`;

  const playerSpriteUrl = `https://play.pokemonshowdown.com/sprites/${
    p1.isShiny ? 'gen5-back-shiny' : 'gen5-back'
  }/${p1.sprite.toLowerCase()}.png`;

  const [enemySprite, playerSprite] = await Promise.all([
    fetchImageBuffer(enemySpriteUrl)
      .then(loadImage)
      .catch(() => loadImage(path.resolve(__dirname, './img/missingno.png'))),
    fetchImageBuffer(playerSpriteUrl)
      .then(loadImage)
      .catch(() => loadImage(path.resolve(__dirname, './img/missingno.png')))
  ]);

  // 1. Draw background
  ctx.drawImage(bgImg, 0, 0, width, height);

  // 2. Draw enemy Pokemon
  const enemyX = width - 30 - 105 - (enemySprite.width / 2);
  const enemyY = Math.max(0, 32 + 80 + 32 - enemySprite.height);
  ctx.drawImage(enemySprite, enemyX, enemyY);

  // 3. Draw player Pokemon
  const playerX = 50 + 23 + 64 - (playerSprite.width / 2);
  const playerY = height - 33 - playerSprite.height;
  ctx.drawImage(playerSprite, playerX, playerY);

  // 4. Draw UI overlay
  ctx.drawImage(uiImg, 0, 0, width, height);

  // 5. Draw HP bars
  function drawHp(current, max, x, y) {
    const ratio = Math.max(0, Math.min(1, current / max));
    const fillWidth = Math.round(ratio * 95);
    let color = '#16bf1d';
    if (ratio <= 0.2) color = '#ee4646';
    else if (ratio <= 0.5) color = '#f8bb34';

    ctx.fillStyle = color;
    ctx.fillRect(x, y, fillWidth, 5);
  }

  drawHp(p2.currentHp, p2.maxHp, 99, 53);
  drawHp(p1.currentHp, p1.maxHp, 376, 218);

  // 6. Draw gender icons
  if (p2.gender === 'M' || p2.gender === 'F') {
    ctx.drawImage(p2.gender === 'M' ? maleIcon : femaleIcon, 26, 58);
  }
  if (p1.gender === 'M' || p1.gender === 'F') {
    ctx.drawImage(p1.gender === 'M' ? maleIcon : femaleIcon, 304, height - 67);
  }

  // 7. Draw text
  ctx.font = '18px "Pokemon Fire Red"';
  ctx.fillStyle = '#000000';
  ctx.textBaseline = 'top';

  // Enemy name & level
  ctx.fillText(p2.name.toUpperCase(), 31, 28);
  ctx.fillText(`Lv.${p2.level}`, 155, 28);

  // Player name & level
  ctx.fillText(p1.name.toUpperCase(), 310, height - 95);
  ctx.fillText(`Lv.${p1.level}`, 430, height - 95);

  // HP numbers (right-aligned)
  const enemyHpText = p2.currentHp <= 0 ? 'FAINTED' : `${p2.currentHp}/${p2.maxHp}`;
  const playerHpText = p1.currentHp <= 0 ? 'FAINTED' : `${p1.currentHp}/${p1.maxHp}`;

  ctx.fillText(enemyHpText, 193 - ctx.measureText(enemyHpText).width, 62);
  ctx.fillText(playerHpText, 470 - ctx.measureText(playerHpText).width, height - 62);

  return canvas.toBuffer('image/png');
}

module.exports = { renderBattleScene };
```

---

## 10. Alternatives

### @napi-rs/canvas

If you encounter system dependency issues (`node-gyp`, Cairo, Pango, giflib):

- **Drop-in replacement** for node-canvas
- Built on Rust/Skia
- Pre-compiled binaries
- No system dependencies required

```bash
npm install @napi-rs/canvas
```

---

## Quick Reference

| Task | Key Points |
|------|-----------|
| **Canvas Setup** | Use `519x288`, set `patternQuality = 'nearest'` |
| **Loading Sprites** | Use axios buffer + `loadImage()`, prefer Showdown PNGs |
| **Fonts** | Call `registerFont()` before `createCanvas()` |
| **HP Bars** | Green >50%, Yellow 20–50%, Red <20% |
| **Layering** | BG → Sprites → UI → HP → Text |
| **Discord** | Use `AttachmentBuilder`, embed with `attachment://` |
| **GIFs** | ⚠️ Avoid — use static PNGs instead |
| **Crisp Sprites** | Set `quality = 'nearest'` and `patternQuality = 'nearest'` |
