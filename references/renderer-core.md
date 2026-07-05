# Renderer Core — HTML, CSS, App Controller (v3 final)

Main window is 56x56, never resizes. Menu and selection are separate windows.
Only canvas + clickTarget in the HTML. No in-window panels.

---

## index.html

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pixel Eye Care Pet</title>
<link rel="stylesheet" href="style.css">
</head>
<body>

<canvas id="petCanvas" width="56" height="56"></canvas>
<div id="clickTarget"></div>

<script src="sprites.js"></script>
<script src="particles.js"></script>
<script src="sound.js"></script>
<script src="canvas.js"></script>
<script src="app.js"></script>
</body>
</html>
```

---

## style.css

```css
* { margin: 0; padding: 0; box-sizing: border-box; }

html, body {
  width: 100%; height: 100%;
  overflow: hidden;
  background: transparent;
  user-select: none;
  -webkit-user-select: none;
}

#petCanvas {
  display: block;
  width: 56px; height: 56px;
  image-rendering: pixelated;
  image-rendering: crisp-edges;
}

#clickTarget {
  position: fixed;
  top: 0; left: 0;
  width: 56px; height: 56px;
  cursor: grab;
  z-index: 500;
}
#clickTarget:active { cursor: grabbing; }
```

---

## app.js

```javascript
// app.js — Application Controller (default pet: whale)
const EYE_CARE_INTERVAL = 20 * 60 * 1000;
const LEISURE_INTERVAL = 5 * 60 * 1000;
const LEISURE_VARIANCE = 30 * 1000;
const LEISURE_DURATION = 5000;
const DANCE_DURATION = 3000;
const CHIME_DURATION = 5000;

const STATE = {
  petType: 'whale', mode: 'idle',
  eyeTimerId: null, leisureTimerId: null, danceTimerId: null,
  isDragging: false, dragStart: { x: 0, y: 0 },
  lastPetPosition: null, selectionMade: false,
  gameMode: false, companionMode: false,
};

const canvas = document.getElementById('petCanvas');
const ctx = canvas.getContext('2d');
const clickTarget = document.getElementById('clickTarget');

let petConfig = null, animationId = null, animTimer = 0;
let currentAnim = null, currentFrame = 0, particleSystem = null;

async function init() {
  particleSystem = new ParticleSystem();
  const savedType = await window.electronAPI.getPetType();
  if (savedType && PETS[savedType]) {
    STATE.petType = savedType; petConfig = PETS[savedType];
  } else {
    STATE.petType = 'whale'; petConfig = PETS['whale'];
    await window.electronAPI.savePetType('whale');
  }
  STATE.selectionMade = true;
  currentAnim = 'idle'; currentFrame = 0; animTimer = 0;
  startApp();

  window.electronAPI.onResetTimer(() => { triggerReset(); });
  window.electronAPI.onReminderDismissed(() => { onReminderDismissed(); });
  window.electronAPI.onSwitchPet(() => { showSelectionPanel(); });
  window.electronAPI.onShowDance(() => { triggerShowDance(); });
  window.electronAPI.onHidePet(() => { hideWithAnimation(); });
  window.electronAPI.onToggleGame(() => { toggleGameMode(); });
  window.electronAPI.onToggleCompanion(() => { toggleCompanionMode(); });
  window.electronAPI.onPetTypeChanged((type) => {
    if (type && PETS[type]) {
      STATE.petType = type; petConfig = PETS[type];
      currentAnim = 'idle'; currentFrame = 0; animTimer = 0;
    }
  });

  const savedPos = await window.electronAPI.getPosition();
  if (savedPos) STATE.lastPetPosition = savedPos;
  setupEventHandlers();
}

function showSelectionPanel() { window.electronAPI.showSelectionPanel(); }
function startApp() { startEyeTimer(); scheduleLeisure(); startRenderLoop(); }

// ── Eye Timer ─────────────────────────────────────────────────────
function startEyeTimer() {
  clearInterval(STATE.eyeTimerId);
  if (STATE.companionMode) return;
  STATE.eyeTimerId = setInterval(() => { triggerEyeReminder(); }, EYE_CARE_INTERVAL);
}

let gameChimeTimer = null;

async function triggerEyeReminder() {
  if (STATE.mode === 'reminder' || STATE.mode === 'dance') return;
  if (STATE.companionMode) return;
  STATE.mode = 'reminder'; currentAnim = 'jump'; currentFrame = 0; animTimer = 0;
  particleSystem.spawnJumpParticles(28, 22);
  await window.electronAPI.showReminder(STATE.gameMode);
  if (STATE.gameMode) {
    playWindChime(CHIME_DURATION);
    gameChimeTimer = setInterval(() => playWindChime(CHIME_DURATION), 7000);
  } else { playWindChime(CHIME_DURATION); }
}

function onReminderDismissed() {
  if (STATE.mode !== 'reminder') return;
  clearInterval(gameChimeTimer); gameChimeTimer = null;
  STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0;
  particleSystem.clear(); startEyeTimer();
}

// ── Leisure Timer ─────────────────────────────────────────────────
function scheduleLeisure() {
  clearTimeout(STATE.leisureTimerId);
  const variance = (Math.random() - 0.5) * 2 * LEISURE_VARIANCE;
  STATE.leisureTimerId = setTimeout(() => {
    if (STATE.mode === 'idle') triggerLeisure();
    scheduleLeisure();
  }, LEISURE_INTERVAL + variance);
}

function triggerLeisure() {
  const anims = ['drink', 'stretch', 'bath', 'eatFruit', 'eatCake', 'massage'];
  STATE.mode = 'leisure'; currentAnim = anims[Math.floor(Math.random() * anims.length)];
  currentFrame = 0; animTimer = 0;
  setTimeout(() => {
    if (STATE.mode === 'leisure') { STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; }
  }, LEISURE_DURATION);
}

// ── Reset: jump + hearts ──────────────────────────────────────────
function triggerReset() {
  clearInterval(STATE.eyeTimerId); clearTimeout(STATE.leisureTimerId);
  clearTimeout(STATE.danceTimerId); clearInterval(gameChimeTimer); gameChimeTimer = null;
  STATE.gameMode = false; STATE.companionMode = false;  // exit special modes
  window.electronAPI.closeReminder();
  STATE.mode = 'resetJump'; currentAnim = 'jump'; currentFrame = 0; animTimer = 0;
  particleSystem.spawnHeartParticles(28, 18);
  STATE.danceTimerId = setTimeout(() => {
    STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0;
    particleSystem.clear(); startEyeTimer(); scheduleLeisure();
  }, 3000);
}

function triggerShowDance() {
  if (STATE.mode === 'reminder') return;
  STATE.mode = 'appearing'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0;
}

function hideWithAnimation() {
  if (STATE.mode === 'hiding') return;
  STATE.mode = 'hiding'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0;
}

// ── Render Loop ───────────────────────────────────────────────────
function startRenderLoop() {
  let lastTime = performance.now();

  function drawBlackHole(progress) {
    const cx = 28, baseY = 48;
    const openAmount = progress <= 0.5 ? progress * 2 : (1 - progress) * 2;
    const w = 6 + openAmount * 26, h = 2 + openAmount * 8;
    ctx.fillStyle = '#1a1a1a';
    ctx.beginPath(); ctx.ellipse(cx, baseY, w / 2, h / 2, 0, 0, Math.PI * 2); ctx.fill();
    ctx.fillStyle = '#000000';
    ctx.beginPath(); ctx.ellipse(cx, baseY, w * 0.4, h * 0.4, 0, 0, Math.PI * 2); ctx.fill();
    ctx.strokeStyle = '#8844aa'; ctx.lineWidth = 1;
    ctx.beginPath(); ctx.ellipse(cx, baseY, w / 2, h / 2, 0, 0, Math.PI * 2); ctx.stroke();
    ctx.lineWidth = 1;
  }

  function loop(now) {
    const dt = now - lastTime; lastTime = now; animTimer += dt;
    ctx.clearRect(0, 0, 70, 70);
    let petOffsetY = 0, petScaleY = 1, petAlpha = 1;

    // Hiding
    if (STATE.mode === 'hiding') {
      const t = animTimer;
      if (t < 400) { drawBlackHole(t / 400 * 0.5); }
      else if (t < 700) { drawBlackHole(0.5); const s = (t - 400) / 300; petOffsetY = s * 40; petScaleY = 1 - s * 0.8; }
      else if (t < 900) { const c = (t - 700) / 200; drawBlackHole(0.5 + c * 0.5); petOffsetY = 40; petScaleY = 0.2; petAlpha = 1 - c; }
      else { ctx.clearRect(0, 0, 70, 70); window.electronAPI.hideWindow(); STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; petAlpha = 0; petOffsetY = 0; petScaleY = 1; }
    }
    // Appearing
    else if (STATE.mode === 'appearing') {
      const t = animTimer;
      if (t < 400) { drawBlackHole(t / 400 * 0.5); petAlpha = 0; }
      else if (t < 700) { drawBlackHole(0.5); const r = (t - 400) / 300; petOffsetY = 40 - r * 40; petScaleY = 0.2 + r * 0.8; petAlpha = r; }
      else if (t < 900) { drawBlackHole(0.5 - (t - 700) / 200 * 0.5); petAlpha = 1; }
      else { STATE.mode = 'dance'; currentAnim = 'dance'; currentFrame = 0; animTimer = 0; particleSystem.spawnDanceParticles(28, 22); }
    }

    // Render hole-mode pet
    if ((STATE.mode === 'hiding' || STATE.mode === 'appearing') && (STATE.mode !== 'hiding' || animTimer < 700)) {
      const frames = petConfig?.animations[currentAnim] || petConfig?.animations.idle;
      if (frames?.length) {
        currentFrame = Math.floor(animTimer / (currentAnim === 'idle' ? 800 : 150)) % frames.length;
        ctx.save(); ctx.imageSmoothingEnabled = false; ctx.globalAlpha = petAlpha;
        if (petScaleY !== 1 || petOffsetY !== 0) { const baseY = 10 + petOffsetY; ctx.translate(0, baseY); ctx.scale(1, petScaleY); ctx.translate(0, -baseY); }
        renderSprite(ctx, frames[currentFrame], petConfig.colors, 3, 3, 1);
        ctx.restore();
      }
    }

    // Dance
    if (STATE.mode === 'dance') {
      const frames = petConfig?.animations.dance;
      if (frames?.length) { renderSprite(ctx, frames[Math.floor(animTimer / 200) % frames.length], petConfig.colors, 3, 3, 1); }
      particleSystem.update(dt / 1000); particleSystem.render(ctx);
      if (animTimer >= DANCE_DURATION) { STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; particleSystem.clear(); }
    }

    // Reset jump + hearts
    if (STATE.mode === 'resetJump') {
      const frames = petConfig?.animations?.jump;
      if (frames?.length && petConfig) { renderSprite(ctx, frames[Math.floor(animTimer / 150) % frames.length], petConfig.colors, 3, 3, 1); }
      particleSystem.update(dt / 1000); particleSystem.render(ctx);
      if (animTimer >= 3000) { STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; particleSystem.clear(); }
    }

    // Game mode animation (jump + music notes)
    if (STATE.mode === 'gameAnim') {
      const frames = petConfig?.animations?.jump || petConfig?.animations?.idle;
      if (frames?.length && petConfig) { renderSprite(ctx, frames[Math.floor(animTimer / 150) % frames.length], petConfig.colors, 3, 3, 1); }
      particleSystem.update(dt / 1000); particleSystem.render(ctx);
      if (animTimer >= 3000) { STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; particleSystem.clear(); }
    }
    // Companion mode animation (closed eyes + Zzz)
    if (STATE.mode === 'companionAnim') {
      const frames = petConfig?.animations?.idle;
      if (frames?.length && petConfig) {
        renderSprite(ctx, frames[0], petConfig.colors, 3, 3, 1);
        ctx.fillStyle = petConfig.colors.outline || '#334455';
        ctx.fillRect(21, 23, 4, 1); ctx.fillRect(31, 23, 4, 1);  // closed eyes
      }
      particleSystem.update(dt / 1000); particleSystem.render(ctx);
      if (animTimer >= 3000) { STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; particleSystem.clear(); }
    }

    // Normal modes
    if (STATE.mode !== 'hiding' && STATE.mode !== 'appearing' && STATE.mode !== 'dance' && STATE.mode !== 'resetJump' && STATE.mode !== 'gameAnim' && STATE.mode !== 'companionAnim') {
      const frames = petConfig?.animations[currentAnim] || petConfig?.animations?.idle;
      if (frames?.length && petConfig) {
        currentFrame = Math.floor(animTimer / (currentAnim === 'idle' ? 800 : 150)) % frames.length;
        ctx.imageSmoothingEnabled = false;
        renderSprite(ctx, frames[currentFrame], petConfig.colors, 3, 3, 1);
      }
      particleSystem.update(dt / 1000); particleSystem.render(ctx);
    }

    // Safety net
    if (!petConfig && STATE.mode !== 'hiding' && STATE.mode !== 'appearing') {
      const whale = PETS?.whale;
      if (whale) { petConfig = whale; currentAnim = 'idle'; ctx.imageSmoothingEnabled = false; renderSprite(ctx, whale.animations.idle[0], whale.colors, 3, 3, 1); }
    }

    animationId = requestAnimationFrame(loop);
  }
  animationId = requestAnimationFrame(loop);
}

// ── Event Handlers ────────────────────────────────────────────────
function setupEventHandlers() {
  clickTarget.addEventListener('mousedown', (e) => {
    if (e.button !== 0 || STATE.mode === 'leisure') return;
    STATE.isDragging = true; STATE.dragStart.x = e.screenX; STATE.dragStart.y = e.screenY;
    document.body.classList.add('dragging');
  });
  document.addEventListener('mousemove', (e) => {
    if (!STATE.isDragging) return;
    const dx = e.screenX - STATE.dragStart.x, dy = e.screenY - STATE.dragStart.y;
    if (Math.abs(dx) < 1 && Math.abs(dy) < 1) return;
    STATE.dragStart.x = e.screenX; STATE.dragStart.y = e.screenY;
    try { window.electronAPI.moveWindow(dx, dy); } catch (_) {}
  });
  document.addEventListener('mouseup', async () => {
    if (!STATE.isDragging) return;
    STATE.isDragging = false; document.body.classList.remove('dragging');
    try { if (STATE.lastPetPosition) await window.electronAPI.savePosition(STATE.lastPetPosition.x, STATE.lastPetPosition.y); } catch (_) {}
  });
  clickTarget.addEventListener('contextmenu', (e) => { e.preventDefault(); window.electronAPI.showContextMenu(); });
}

function toggleGameMode() {
  STATE.gameMode = !STATE.gameMode;
  if (STATE.gameMode) {
    STATE.companionMode = false;  // mutually exclusive
    clearInterval(STATE.eyeTimerId); startEyeTimer();  // restart timer
    if (STATE.mode === 'idle') {
      STATE.mode = 'gameAnim'; currentAnim = 'jump'; currentFrame = 0; animTimer = 0;
      particleSystem.spawnMusicNotes(28, 15);
      setTimeout(() => { if (STATE.mode === 'gameAnim') { STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; particleSystem.clear(); } }, 3000);
    }
  }
}
function toggleCompanionMode() {
  STATE.companionMode = !STATE.companionMode;
  if (STATE.companionMode) {
    STATE.gameMode = false;  // mutually exclusive
    clearInterval(gameChimeTimer); gameChimeTimer = null;
  }
  clearInterval(STATE.eyeTimerId);
  if (!STATE.companionMode) startEyeTimer();
  if (STATE.companionMode && STATE.mode === 'idle') {
    STATE.mode = 'companionAnim'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0;
    particleSystem.spawnSleepZ(28, 25);
    setTimeout(() => { if (STATE.mode === 'companionAnim') { STATE.mode = 'idle'; currentAnim = 'idle'; currentFrame = 0; animTimer = 0; particleSystem.clear(); } }, 3000);
  }
}

init();
```

---

## Additional renderer files

- **menu.html** — Pixel context menu (separate window, 130x155) — see `electron-core.md`
- **selection.html** — Pet selection grid (separate window, 240x280) — see `electron-core.md`
- **reminder.html** — Eye care reminder dialog — see `electron-core.md`
- **sprites.js / particles.js / sound.js / canvas.js** — see `pixel-engine.md`
