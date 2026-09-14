Research Report: Building a Pokemon Battle Scene Renderer with node-canvas
Here is the complete technical breakdown for using canvas (node-canvas) in Node.js to render Pokemon battle scenes, complete with code examples, sprite compositing rules, Discord.js integration, and answers to all 8 specific questions.

1. How to Create a Canvas and Draw on it in Node.js
In node-canvas, the HTML5 Canvas API is replicated using the Cairo graphics engine.

javascript

const { createCanvas } = require('canvas');
// Dimensions matching standard Pokemon Showdown battle scenes (518x288)
const width = 518;
const height = 288;
const canvas = createCanvas(width, height);
const ctx = canvas.getContext('2d');
// Configure crisp rendering for pixel art (Cairo specific)
ctx.patternQuality = 'nearest';
ctx.quality = 'nearest';
// Basic background fill
ctx.fillStyle = '#202020';
ctx.fillRect(0, 0, width, height);
2. Loading Images (PNG & GIF) with loadImage
Supported Formats & Sources
loadImage() takes local file paths, remote URLs, data URIs, or Buffer instances and returns a Promise resolving to an Image:

javascript

const { loadImage } = require('canvas');
const axios = require('axios');
// 1. Local path
const bg = await loadImage('./assets/bgs/forest.jpeg');
// 2. Buffer (recommended for remote network requests)
const response = await axios.get('https://play.pokemonshowdown.com/sprites/gen5/charizard.png', {
  responseType: 'arraybuffer',
  timeout: 5000
});
const sprite = await loadImage(Buffer.from(response.data));
Does loadImage support GIF? Does it render the first frame?
Yes, loadImage supports GIF decoding via giflib.
It only parses and draws the first frame (Frame 0).
It does not animate or cycle through frames. When drawn with ctx.drawImage(), it behaves as a static image.
3. Drawing Text with Custom TTF Fonts (registerFont)
registerFont registers external .ttf or .otf font files with Cairo/Pango before rendering.

javascript

const { registerFont, createCanvas } = require('canvas');
const path = require('path');
// 1. Register font BEFORE drawing (must provide family name)
registerFont(path.resolve(__dirname, './img/pokemon_fire_red.ttf'), {
  family: 'Pokemon Fire Red'
});
// 2. Use in context
const canvas = createCanvas(518, 288);
const ctx = canvas.getContext('2d');
ctx.font = '18px "Pokemon Fire Red"';
ctx.fillStyle = '#000000';
ctx.textBaseline = 'top';
// 3. Drop shadow effect (standard in Pokemon UI)
const text = 'PIKACHU';
const x = 31, y = 28;
ctx.fillStyle = '#686868'; // Shadow
ctx.fillText(text, x + 1, y + 1);
ctx.fillStyle = '#202020'; // Main text
ctx.fillText(text, x, y);
// 4. Measure text for right-alignment (e.g., HP numbers)
const hpText = '120/150';
const textWidth = ctx.measureText(hpText).width;
ctx.fillText(hpText, 470 - textWidth, 226);
Rules & Gotchas for registerFont:

Call registerFont() before invoking createCanvas() or setting ctx.font.
Always use path.resolve() or absolute paths.
The family string must be wrapped in quotes inside ctx.font = '18px "My Font"'.
4. Drawing Rectangles (Filled) for HP Bars
An HP bar consists of:

An empty/background bar (dark gray/black track)
A filled bar proportional to current HP / max HP
Dynamic color changes: Green (>50%), Yellow (20%–50%), Red (<20%)
javascript

function drawHpBar(ctx, currentHp, maxHp, x, y, maxWidth = 95, height = 5) {
  const hpRatio = Math.max(0, Math.min(1, currentHp / maxHp));
  const currentWidth = Math.round(hpRatio * maxWidth);
  // 1. Empty / background track
  ctx.fillStyle = '#404040';
  ctx.fillRect(x, y, maxWidth, height);
  // 2. Determine color threshold
  let barColor = '#16bf1d'; // Green
  if (hpRatio <= 0.20) {
    barColor = '#ee4646';   // Red
  } else if (hpRatio <= 0.50) {
    barColor = '#f8bb34';   // Yellow / Orange
  }
  // 3. Draw active HP fill
  if (currentWidth > 0) {
    ctx.fillStyle = barColor;
    ctx.fillRect(x, y, currentWidth, height);
  }
}
// Coordinates matching Pokemon Showdown / FireRed HUD (for 518x288 canvas):
// Enemy HP bar:
drawHpBar(ctx, enemyCurrentHp, enemyMaxHp, 99, 53, 95, 5);
// Player HP bar:
drawHpBar(ctx, playerCurrentHp, playerMaxHp, 376, 218, 95, 5);
5. Compositing & Layering Battle Sprites at Coordinates
Layer Order:
Background image (e.g. forest.jpeg at 0, 0)
Enemy Pokemon Sprite (top-right platform area)
Player Pokemon Sprite (bottom-left area)
UI Overlay / HUD (ui.png at 0, 0 — contains HP boxes and text containers)
Dynamic HP Bars
Gender Icons & Status Ailments (BRN, PAR, etc.)
Text (Names, Levels, HP numbers)
Coordinate Mathematics (518x288 Canvas):
javascript

// Enemy Sprite (Top-Right):
const enemyX = 518 - 30 - 105 - (enemySprite.width / 2);
const enemyY = Math.max(0, 32 + 80 + 32 - enemySprite.height);
ctx.drawImage(enemySprite, enemyX, enemyY);
// Player Sprite (Bottom-Left):
const playerX = 50 + 23 + 64 - (playerSprite.width / 2);
const playerY = 288 - 33 - playerSprite.height;
ctx.drawImage(playerSprite, playerX, playerY);
Horizontally Flipping a Front Sprite (if Back Sprite is missing):
javascript

function drawFlippedSprite(ctx, image, x, y) {
  ctx.save();
  ctx.translate(x + image.width, y);
  ctx.scale(-1, 1);
  ctx.drawImage(image, 0, 0);
  ctx.restore();
}
6. Exporting Canvas as a PNG Buffer to Discord.js (v14)
In node-canvas, canvas.toBuffer('image/png') is synchronous and returns a native Node Buffer:

javascript

const { AttachmentBuilder, EmbedBuilder } = require('discord.js');
// 1. Export as PNG Buffer
const buffer = canvas.toBuffer('image/png');
// 2. Wrap in Discord AttachmentBuilder
const attachment = new AttachmentBuilder(buffer, { name: 'battle.png' });
// 3. Option A: Direct Attachment
await interaction.reply({
  content: 'A wild battle started!',
  files: [attachment]
});
// 4. Option B: Attached inside an Embed
const embed = new EmbedBuilder()
  .setTitle('Wild Pokémon Encounter!')
  .setColor(0x5865F2)
  .setImage('attachment://battle.png');
await interaction.reply({
  embeds: [embed],
  files: [attachment]
});
7. Handling Transparency & Alpha when Pasting Sprites
Default Alpha Blending: node-canvas defaults to ctx.globalCompositeOperation = 'source-over'. When drawing PNGs with transparent or semi-transparent alpha channels, the background shows through cleanly.
Do NOT clear with black: If you initialize the canvas, it starts transparent (rgba(0, 0, 0, 0)).
Pixel Crispness (Preventing Blurry Sprites): By default, scaling images applies bilinear smoothing. For pixel art Pokemon sprites:
javascript

// In node-canvas (Cairo):
ctx.patternQuality = 'nearest';
ctx.quality = 'nearest';
(Note: ctx.imageSmoothingEnabled = false is standard in browsers, but node-canvas uses ctx.patternQuality = 'nearest' to enable nearest-neighbor scaling).
8. Gotchas with GIF Loading in node-canvas
Only First Frame is Rendered: loadImage() will never animate. It decodes and stays on frame 0.
Frame 0 Delta / Disposal Artifacts: Many animated GIFs from Pokemon Showdown or web scrapers use inter-frame delta compression or disposal methods ("restore to background", "restore to previous"). If frame 0 is an empty transparent frame or only contains a partial character limb before subsequent frames fill it in, loadImage() will render a corrupt or partially drawn sprite.
Black Background Bug on Transparent GIFs: In certain builds of libgif, transparent palette indices in 1-bit transparency GIFs get converted into solid black #000000 pixels instead of alpha 0.
Network Timeout / Hangs: Using loadImage('https://...') directly uses Cairo's internal HTTP client which lacks connection timeouts, custom User-Agents, or retry logic. If the server hangs or blocks requests, the bot process will hang. Always fetch via axios / node-fetch with a timeout, then pass the Buffer to loadImage(buf).
Showdown Has Native Static PNG Repositories: Instead of loading GIFs and hoping frame 0 isn't corrupted, Pokemon Showdown maintains static PNG directories specifically for this:
Front Sprites: https://play.pokemonshowdown.com/sprites/gen5/{name}.png
Shiny Front: https://play.pokemonshowdown.com/sprites/gen5-shiny/{name}.png
Back Sprites: https://play.pokemonshowdown.com/sprites/gen5-back/{name}.png
Shiny Back: https://play.pokemonshowdown.com/sprites/gen5-back-shiny/{name}.png
PokeAPI fallback: https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/{id}.png
Complete Reusable Battle Scene Renderer Function
javascript

const { createCanvas, loadImage, registerFont } = require('canvas');
const axios = require('axios');
const path = require('path');
// 1. Register fonts once at module load
registerFont(path.resolve(__dirname, './img/pokemon_fire_red.ttf'), {
  family: 'Pokemon Fire Red'
});
async function fetchImageBuffer(url) {
  const res = await axios.get(url, { responseType: 'arraybuffer', timeout: 5000 });
  return Buffer.from(res.data);
}
/**
 * Renders a Pokemon battle scene and returns a PNG buffer.
 */
async function renderBattleScene({
  bgLocation = 'town',
  p1 = { name: 'Charizard', level: 50, currentHp: 150, maxHp: 150, gender: 'M', sprite: 'charizard', isShiny: false },
  p2 = { name: 'Pikachu', level: 48, currentHp: 80, maxHp: 100, gender: 'F', sprite: 'pikachu', isShiny: false }
}) {
  const width = 518;
  const height = 288;
  const canvas = createCanvas(width, height);
  const ctx = canvas.getContext('2d');
  // Nearest-neighbor scaling for crisp retro pixel art
  ctx.patternQuality = 'nearest';
  // Load static assets
  const [bgImg, uiImg, maleIcon, femaleIcon] = await Promise.all([
    loadImage(path.resolve(__dirname, `./img/bgs/${bgLocation}.jpeg`)).catch(() =>
      loadImage(path.resolve(__dirname, './img/bgs/-1.jpeg'))
    ),
    loadImage(path.resolve(__dirname, './img/ui.png')),
    loadImage(path.resolve(__dirname, './img/male.png')),
    loadImage(path.resolve(__dirname, './img/female.png'))
  ]);
  // Load Pokemon sprites (prefer Showdown static PNGs)
  const enemySpriteUrl = `https://play.pokemonshowdown.com/sprites/${p2.isShiny ? 'gen5-shiny' : 'gen5'}/${p2.sprite.toLowerCase()}.png`;
  const playerSpriteUrl = `https://play.pokemonshowdown.com/sprites/${p1.isShiny ? 'gen5-back-shiny' : 'gen5-back'}/${p1.sprite.toLowerCase()}.png`;
  const [enemySprite, playerSprite] = await Promise.all([
    fetchImageBuffer(enemySpriteUrl).then(loadImage).catch(() => loadImage(path.resolve(__dirname, './img/missingno.png'))),
    fetchImageBuffer(playerSpriteUrl).then(loadImage).catch(() => loadImage(path.resolve(__dirname, './img/missingno.png')))
  ]);
  // 1. Draw Background
  ctx.drawImage(bgImg, 0, 0, width, height);
  // 2. Draw Enemy Pokemon
  const enemyX = width - 30 - 105 - (enemySprite.width / 2);
  const enemyY = Math.max(0, 32 + 80 + 32 - enemySprite.height);
  ctx.drawImage(enemySprite, enemyX, enemyY);
  // 3. Draw Player Pokemon
  const playerX = 50 + 23 + 64 - (playerSprite.width / 2);
  const playerY = height - 33 - playerSprite.height;
  ctx.drawImage(playerSprite, playerX, playerY);
  // 4. Draw UI Frame overlay
  ctx.drawImage(uiImg, 0, 0, width, height);
  // 5. Draw Dynamic HP Bars (width = 95, height = 5)
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
  // 6. Draw Genders
  if (p2.gender === 'M' || p2.gender === 'F') {
    ctx.drawImage(p2.gender === 'M' ? maleIcon : femaleIcon, 26, 58);
  }
  if (p1.gender === 'M' || p1.gender === 'F') {
    ctx.drawImage(p1.gender === 'M' ? maleIcon : femaleIcon, 304, height - 67);
  }
  // 7. Draw Text
  ctx.font = '18px "Pokemon Fire Red"';
  ctx.fillStyle = '#000000';
  ctx.textBaseline = 'top';
  // Enemy Name & Level
  ctx.fillText(p2.name.toUpperCase(), 31, 28);
  ctx.fillText(`Lv.${p2.level}`, 155, 28);
  // Player Name & Level
  ctx.fillText(p1.name.toUpperCase(), 310, height - 95);
  ctx.fillText(`Lv.${p1.level}`, 430, height - 95);
  // HP Numbers (Right aligned)
  const enemyHpText = p2.currentHp <= 0 ? 'FAINTED' : `${p2.currentHp}/${p2.maxHp}`;
  const playerHpText = p1.currentHp <= 0 ? 'FAINTED' : `${p1.currentHp}/${p1.maxHp}`;
  ctx.fillText(enemyHpText, 193 - ctx.measureText(enemyHpText).width, 62);
  ctx.fillText(playerHpText, 470 - ctx.measureText(playerHpText).width, height - 62);
  return canvas.toBuffer('image/png');
}
module.exports = { renderBattleScene };
Modern Alternative Note: @napi-rs/canvas
If you encounter system dependency compilation issues (node-gyp, Cairo, Pango, or giflib errors on deployment/Docker), @napi-rs/canvas is a drop-in replacement built on Rust/Skia that ships pre-compiled binaries for all platforms with zero C++ compilation steps and uses the exact same API.
