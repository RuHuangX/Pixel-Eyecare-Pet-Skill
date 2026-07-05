---
name: pixel-eye-care-pet
description: >
  Generate a complete pixel-art desktop eye-care pet application based on the
  20-20-20 rule. This skill creates a full Electron project with a draggable
  desktop companion that reminds users to rest their eyes every 20 minutes.
  Use this skill whenever the user asks for a desktop pet, eye care reminder,
  pomodoro-style health timer, pixel art companion, screen break reminder,
  QQ-pet-style desktop mascot, or any kind of cute desktop widget for health
  reminders. Even if the user doesn't mention "Electron" or "desktop app"
  explicitly, if they describe a floating desktop character or eye-break
  timer, this is the right skill.
  中文匹配关键词：护眼桌面宠物, 20-20-20规则, 桌面小宠物, 健康提醒, 休息眼睛, 桌面小精灵,桌面小动物. 
---

# Pixel Eye Care Pet — 20-20-20 护眼桌面宠物

## Overview

This skill generates a complete Electron desktop application: a draggable
pixel-art pet that lives on the user's desktop and reminds them to follow
the **20-20-20 rule** (every 20 minutes, look at something 20 feet away for
20 seconds). The pet also performs random leisure animations between
reminders, and supports 6 selectable pixel-art animal companions.

## When and How to Use This Skill

When triggered, you (Claude) will create EVERY file needed for a complete,
runnable Electron project. Follow the step-by-step instructions below in
order. Read the reference files as directed — they contain complete code
templates.

**Before writing any files, ask the user:**
1. Where should the project be created? (default: `./pixel-eye-care-pet/`
   in the current working directory)
2. Which pet would they like? (鲸鱼/章鱼/兔子/猫咪/小狗/仓鼠, default: 猫咪)

If they don't specify, use the defaults and proceed.

## Architecture Overview

```
project-root/
├── package.json          # Electron + build config
├── main.js               # Electron main process (window, tray, IPC)
├── preload.js            # Context bridge for IPC
├── icon.png              # Pixel eye icon (16×16 to 256×256, PNG)
├── icon.ico              # Windows icon (multi-res from icon.png)
├── renderer/
│   ├── index.html        # Main window HTML (pet + dialogs)
│   ├── style.css         # Pixel-art styling
│   ├── app.js            # App controller (timers, state, IPC)
│   ├── canvas.js         # Pixel sprite renderer (Canvas 2D)
│   ├── sprites.js        # All pixel-art sprite definitions
│   ├── particles.js      # Particle effect system
│   ├── sound.js          # Web Audio API chime generator
│   └── selection.js      # First-run pet selection panel
├── build/                # (optional) electron-builder config
└── README.md             # Usage & deployment guide
```

**Technology**: Electron + vanilla HTML/CSS/JS + Canvas 2D. No frameworks
needed — keeps the dependency footprint minimal and startup fast.

**Window**: 56×56 px frameless, transparent, draggable, always-on-top. Smart
fullscreen detection: auto-hides when a fullscreen app/game is detected,
auto-restores when fullscreen exits. Renders the 50×50 px pet centered with a
3px margin. Uses a `#clickTarget` div for precise hit testing.

**Reminder dialog**: Centered 300×200 px pixel-styled popup window.

---

## Step-by-Step Implementation

### Step 0: Read the bundled reference files

Before writing code, read these references for complete code templates:

1. `references/electron-core.md` — main.js, preload.js, package.json
2. `references/renderer-core.md` — index.html, style.css, app.js
3. `references/pixel-engine.md` — canvas.js, sprites.js, particles.js, sound.js

Read each one before writing the corresponding files. The reference files
contain production-ready code; adapt variable names and paths to match the
user's project directory.

---

### Step 1: Create package.json

```json
{
  "name": "pixel-eye-care-pet",
  "version": "1.0.0",
  "description": "像素风20-20-20护眼桌面宠物",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "build:win": "electron-builder --win portable",
    "build:mac": "electron-builder --mac",
    "build:linux": "electron-builder --linux"
  },
  "build": {
    "appId": "com.pixelpet.eyecare",
    "productName": "像素护眼宠物",
    "win": {
      "icon": "icon.png",
      "target": "portable"
    },
    "mac": {
      "icon": "icon.png",
      "target": "dmg"
    },
    "linux": {
      "icon": "icon.png",
      "target": "AppImage"
    },
    "files": ["main.js", "preload.js", "renderer/**", "icon.png"]
  },
  "devDependencies": {
    "electron": "^28.0.0",
    "electron-builder": "^24.0.0"
  }
}
```

### Step 2: Create main.js — Electron Main Process

Key responsibilities:
- Create frameless, transparent, always-on-top BrowserWindow (70×70 px)
- Position near bottom-right of screen (or restore saved position)
- Create system tray with pixel eye icon and right-click "退出" menu
- Handle IPC: `save-position`, `get-position`, `show-reminder`, `close-reminder`
- Use `electron-store`-like pattern (or JSON file) to persist pet position
  and selected pet type across launches

**Critical details:**
- Window flags: `transparent: true, frame: false, alwaysOnTop: true, resizable: false, skipTaskbar: true`
- Enable `nodeIntegration` via preload script, NOT directly
- **System tray is mandatory**: Import `Tray` from electron, create a tray instance with the pixel eye icon, and attach a context menu with at minimum a "Quit"/"退出" option. Without this, users cannot cleanly exit the app since there's no taskbar button.
- Tray icon: Use the pixel eye icon (from `nativeImage.createFromDataURL()` or the generated `icon.png`). If the icon file doesn't exist, generate it programmatically as a fallback.
- On window `close` event: hide to tray instead of quitting (unless `app.isQuitting`)

Read `references/electron-core.md` for the complete main.js implementation.

### Step 3: Create preload.js

Expose safe IPC methods to the renderer via `contextBridge`:
- `window.electronAPI.savePosition(x, y)`
- `window.electronAPI.getPosition()` → Promise<{x, y}>
- `window.electronAPI.savePetType(type)` — persist chosen pet
- `window.electronAPI.getPetType()` → Promise<string>
- `window.electronAPI.minimizeToTray()`
- `window.electronAPI.onResetTimer(callback)` — listen from tray menu

### Step 4: Create renderer/index.html

A single HTML file that contains:
- `<canvas id="petCanvas">` — 70×70, renders the pet sprite + particles
- `<div id="reminderDialog">` — hidden by default, shown on timer fire
- `<div id="contextMenu">` — custom right-click menu
- `<div id="selectionPanel">` — first-run pet selection (6 cards)

Keep the markup minimal. All rendering is done on canvas.

### Step 5: Create renderer/style.css

Pixel-art aesthetic throughout:
- Use `image-rendering: pixelated` on canvas
- Use pixel-style fonts (or fall back to monospace with small size)
- Reminder dialog: border with `border-image` or box-shadow pixel pattern
- Use CSS `@font-face` or system monospace for pixel text feel
- Context menu: pixelated border, no anti-aliasing
- Selection panel: grid of 3×2 pet cards with hover effects

Use these pixel-art CSS patterns:
```css
.pixel-border {
  border: 4px solid transparent;
  border-image: url('data:image/png;base64,...') 4 stretch;
  /* OR use box-shadow technique: */
  box-shadow: 4px 0 0 #5a3a2a, -4px 0 0 #5a3a2a, 0 4px 0 #5a3a2a, 0 -4px 0 #5a3a2a;
}
```

Read `references/renderer-core.md` for the complete CSS.

### Step 6: Create renderer/sprites.js — Pixel Art Definitions

This is the heart of the visual experience. Define all 6 pets with full
animation sets using compact pixel grids.

**Data format**: Each 50×50 sprite is a 50-character-wide, 50-line string
grid. Each character represents one pixel:

| Char | Meaning |
|------|---------|
| ` ` (space) | Transparent |
| `1` | Primary color (body) |
| `2` | Secondary color (details/highlights) |
| `3` | Accent color (eyes/cheeks/accessories) |
| `0` | Outline/shadow |

Each pet is an object:
```javascript
const PET_CAT = {
  name: '猫咪',
  nameEn: 'cat',
  colors: { primary: '#ffcc99', secondary: '#ff9966', accent: '#333333', outline: '#885544' },
  animations: {
    idle:       [FRAME_GRID_1, FRAME_GRID_2],  // 2 frames, loop
    jump:       [FRAME_GRID_1, FRAME_GRID_2, FRAME_GRID_3, FRAME_GRID_4], // 4 frames
    drink:      [/* ... */],  // coffee drinking
    stretch:    [/* ... */],  // stretching
    bath:       [/* ... */],  // bathing
    eatFruit:   [/* ... */],  // eating fruit
    eatCake:    [/* ... */],  // eating cake
    massage:    [/* ... */],  // massage/rolling
    dance:      [/* ... */],  // dance reset
  }
};
```

**All 6 pets**: cat (猫咪), dog (小狗), rabbit (兔子), hamster (仓鼠),
octopus (章鱼), whale (鲸鱼).

**Particle sprites** (used with jump and dance animations):
- Star: 8×8 pixel grid (for jump)
- Flower: 10×10 pixel grid (for dance reset)
- Leaf: 8×10 pixel grid (for dance reset)

Read `references/pixel-engine.md` for the complete sprite definitions with
all 6 pets and all animation frames (pre-drawn compact grids). If the
reference file is unavailable, you MUST draw each pet frame by hand as
compact grids — do NOT skip or use placeholder sprites. Each sprite should
be a recognizable, cute pixel-art animal. Spend the time to make them
charming — this is the most visible part of the application.

### Step 7: Create renderer/canvas.js — Sprite Renderer

A Canvas 2D rendering engine that:
1. Takes a sprite grid definition (50-line string array) and a color map
2. Renders it to the provided canvas context at the correct scale
3. Handles animation frame timing (requestAnimationFrame loop)
4. Draws particles on top

```javascript
// Core rendering function
function renderSprite(ctx, sprite, colors, x, y, scale = 1) {
  const rows = sprite.trim().split('\n');
  for (let row = 0; row < rows.length; row++) {
    for (let col = 0; col < rows[row].length; col++) {
      const ch = rows[row][col];
      if (ch === ' ') continue; // transparent
      const colorMap = { '0': colors.outline, '1': colors.primary, '2': colors.secondary, '3': colors.accent };
      ctx.fillStyle = colorMap[ch] || colors.primary;
      ctx.fillRect(x + col * scale, y + row * scale, scale, scale);
    }
  }
}
```

The render loop should:
- Clear canvas each frame
- Draw current pet animation frame
- Draw active particles (from particle system)
- Advance animation timer
- Use `requestAnimationFrame` for smooth rendering

### Step 8: Create renderer/particles.js — Particle Effects

A simple particle system:
```javascript
class ParticleSystem {
  constructor() {
    this.particles = [];
  }

  // Spawn particles around the pet for jump effect
  spawnJumpParticles(cx, cy) { /* stars that rise and fade */ }

  // Spawn flower/leaf particles for dance/reset effect
  spawnDanceParticles(cx, cy) { /* flowers and leaves appear and fade */ }

  update(dt) { /* move, age, and remove dead particles */ }

  render(ctx) { /* draw all active particles */ }
}
```

Particles should be small (4-8 px), pixel-art shapes with:
- Rising motion (jump particles drift upward)
- Alpha fade-out
- Random spread in x and y
- Duration: ~1.5 seconds per particle

### Step 9: Create renderer/sound.js — Chime Generator

Use the Web Audio API to generate a gentle wind-chime sound programmatically
(no external audio files needed):

```javascript
function playWindChime(durationMs = 5000) {
  const ctx = new (window.AudioContext || window.webkitAudioContext)();

  // Generate 5-8 chime notes with random delays
  const notes = [523, 659, 784, 880, 1047, 1175, 1319]; // C5-E6 pentatonic-ish
  const chimeCount = 7;

  for (let i = 0; i < chimeCount; i++) {
    const delay = Math.random() * durationMs * 0.7;
    const freq = notes[Math.floor(Math.random() * notes.length)];
    setTimeout(() => playChime(ctx, freq, 0.3, 1.5), delay);
  }

  // Auto-stop after duration
  setTimeout(() => ctx.close(), durationMs + 500);
}

function playChime(ctx, frequency, volume, duration) {
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = 'sine';
  osc.frequency.value = frequency;
  gain.gain.setValueAtTime(volume, ctx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + duration);
  osc.connect(gain);
  gain.connect(ctx.destination);
  osc.start(ctx.currentTime);
  osc.stop(ctx.currentTime + duration);
}
```

### Step 10: Create renderer/app.js — Application Controller

The central controller that ties everything together:

**State management:**
```javascript
const STATE = {
  petType: 'cat',          // Selected pet
  mode: 'idle',            // 'idle' | 'reminder' | 'leisure' | 'dance'
  eyeTimer: null,          // 20-min interval ID
  leisureTimer: null,      // 10-min interval ID
  animationFrame: 0,       // Current sprite frame index
  isDragging: false,
  dragStart: { x: 0, y: 0 },
};
```

**Timer logic:**
1. On startup, begin 20-minute eye-care countdown (invisible)
2. Every ~10 minutes (±30s random), trigger a random leisure animation
3. When eye timer fires: show reminder dialog, play chime, animate jump
4. On "确认" click: hide dialog, reset eye timer, resume idle
5. On "重新开始" (right-click): play dance animation, reset ALL timers

**Drag handling:**
- mousedown on canvas: start drag tracking
- mousemove: send window position via IPC (throttled to ~30fps)
- mouseup: save final position via IPC

**IPC listeners:**
- `onResetTimer`: triggered from tray menu or right-click menu

**Selection panel:**
- On first launch (no saved pet type), show selection grid
- On selection: save pet type, hide panel, start app

### Step 11: Create the Reminder Dialog

A separate Electron BrowserWindow, or a positioned div within the main
window. Use a centered overlay approach:

- 300×200 px (scaled from 100×260 as specified, for readability)
- Pixel border styling
- Large pixel-style text: "该休息啦！"
- Subtitle: "向 20 英尺（约 6 米）外远眺 20 秒哦～"
- Pixel button: "确认" (OK)
- Trigger: eye timer fires → show dialog at screen center → wait for click

### Step 12: Create the Right-Click Context Menu

Custom HTML/CSS context menu (not native, to maintain pixel style):
- "重新开始" → triggers dance animation + timer reset
- "切换形象" → reopens pet selection panel
- "退出" → sends quit IPC to main process

Show on `contextmenu` event on the pet canvas. Hide on click outside.

### Step 13: Create the Pixel Eye Icon

Generate a simple but cute pixel-art eye as the application icon. Create a
script to generate `icon.png` (256×256) from pixel data:

```javascript
// icon-generator.js — generates icon.png using canvas
// In the skill, embed this as base64 or generate it inline
```

The icon should be: a large pixel-art eye with iris, pupil, and small
sparkle highlight. Use warm, inviting colors (browns, soft blues, warm
white). Generate at least 16×16, 32×32, 48×48, 64×64, 128×128, 256×256
resolutions and embed as a multi-res PNG or ICO.

Since we may not have native image libraries, provide a data URL of a
pre-designed pixel eye PNG embedded directly in the code. Read
`references/pixel-engine.md` for the pixel eye design.

### Step 14: Create README.md

**Always create a README.md file** — this is essential documentation that every
project needs. Without it, users won't know how to install or run the app.

Write a friendly bilingual (Chinese + English) README:
- What this app does (20-20-20 eye care)
- How to install: `npm install && npm start`
- How to use: double-click icon → pet appears → automatic timers
- How to change pets: right-click → 切换形象
- How to exit: system tray → right-click → 退出
- How to build: `npm run build:win`
- Requirements: Node.js 18+, npm

Name the file `README.md` exactly — this is the standard convention that
GitHub, npm, and most developers expect.

---

## Customization Points

When generating the app, offer these customization hooks (as comments in
the generated code):

1. **Timer durations**: `EYE_CARE_INTERVAL` (default 20 min),
   `LEISURE_INTERVAL` (default 10 min), `LEISURE_VARIANCE` (default 30s)
2. **Reminder message**: Change the dialog text
3. **Sound**: Toggle on/off, change duration
4. **Window size**: Pet scale multiplier

## Quality Checklist

Before presenting the finished project to the user, verify:

- [ ] `package.json` is valid JSON with correct Electron dependency
- [ ] `main.js` handles window creation, tray, IPC, and position persistence
- [ ] `preload.js` exposes only safe API methods
- [ ] All 6 pet sprite sets are present with unique, recognizable designs
- [ ] Each pet has: idle (2+ frames), jump (4 frames), 6 leisure
      animations (3+ frames each), dance (4+ frames)
- [ ] Canvas renderer draws sprites correctly at 50×50 with proper colors
- [ ] Particle system spawns and renders correctly
- [ ] Sound system plays chime on reminder without external files
- [ ] 20-min eye timer works silently, triggers reminder on schedule
- [ ] ~10-min leisure timer works with random variance
- [ ] Right-click menu works with "重新开始", "切换形象", "退出"
- [ ] Pet selection panel shows all 6 options on first launch
- [ ] Drag moves the window smoothly
- [ ] Position is saved and restored across launches
- [ ] System tray icon, right-click shows "退出"
- [ ] Reminder dialog centered, requires click to dismiss
- [ ] Reminder chime stops after 5 seconds
- [ ] README covers install, usage, customization

---

## Reference Files

Read these in order as needed during implementation:

- `references/electron-core.md` — Complete main.js, preload.js code
- `references/renderer-core.md` — Complete HTML, CSS, app.js code
- `references/pixel-engine.md` — Complete sprites, canvas renderer,
  particles, sound, and icon code

Each reference contains production-ready code. If a reference is missing or
unreadable, generate the code from the specifications in this SKILL.md.
