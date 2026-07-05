# Pixel Engine — Sprites, Canvas Renderer, Particles, Sound

Complete rendering engine for the eye-care pet (v2).
All code is production-ready — copy verbatim into the renderer/ directory.

---

## canvas.js

```javascript
// canvas.js — Pixel sprite renderer using Canvas 2D

function renderSprite(ctx, rects, colors, offsetX, offsetY, scale) {
  ctx.imageSmoothingEnabled = false;
  for (const r of rects) {
    let hex = colors[r.c];
    if (!hex && r.c[0] === '#') hex = r.c;
    if (!hex) hex = colors.primary || '#ffcc99';
    ctx.fillStyle = hex;
    ctx.fillRect(
      offsetX + r.x * scale,
      offsetY + r.y * scale,
      r.w * scale,
      r.h * scale
    );
  }
}
```

---

## particles.js

```javascript
// particles.js — Particle effect system

class ParticleSystem {
  constructor() { this.particles = []; }

  spawnJumpParticles(cx, cy) {
    for (let i = 0; i < 12; i++) {
      this.particles.push({
        x: cx + (Math.random() - 0.5) * 20, y: cy + (Math.random() - 0.5) * 10,
        vx: (Math.random() - 0.5) * 30, vy: -20 - Math.random() * 40,
        life: 1.0, decay: 0.6 + Math.random() * 0.3,
        size: 2 + Math.floor(Math.random() * 3),
        color: Math.random() < 0.5 ? '#ffe066' : '#ffdd44', shape: 'star',
      });
    }
  }

  spawnHeartParticles(cx, cy) {
    for (let i = 0; i < 15; i++) {
      this.particles.push({
        x: cx + (Math.random() - 0.5) * 24,
        y: cy + (Math.random() - 0.5) * 10,
        vx: (Math.random() - 0.5) * 15,
        vy: -30 - Math.random() * 50,
        life: 1.0,
        decay: 0.25 + Math.random() * 0.3,
        size: 3 + Math.floor(Math.random() * 3),
        color: ['#ff6b8a', '#ff3366', '#ff8fab', '#ffb3c6'][Math.floor(Math.random() * 4)],
        shape: 'heart',
      });
    }
  }

  spawnMusicNotes(cx, cy) {
    const notes = ['#ffdd00','#ffaa00','#ff8800','#ffcc33','#ffe066'];
    for (let i = 0; i < 12; i++) {
      this.particles.push({
        x: cx + (Math.random() - 0.5) * 30, y: cy + (Math.random() - 0.5) * 20,
        vx: (Math.random() - 0.5) * 25, vy: -25 - Math.random() * 45,
        life: 1.0, decay: 0.25 + Math.random() * 0.35,
        size: 3 + Math.floor(Math.random() * 3),
        color: notes[Math.floor(Math.random() * notes.length)], shape: 'note',
      });
    }
  }
  spawnSleepZ(cx, cy) {
    for (let i = 0; i < 8; i++) {
      this.particles.push({
        x: cx + 10 + Math.random() * 20, y: cy - 10 - Math.random() * 20,
        vx: 5 + Math.random() * 10, vy: -15 - Math.random() * 20,
        life: 1.0, decay: 0.2 + Math.random() * 0.3,
        size: 3 + Math.floor(Math.random() * 3),
        color: '#aaccff', shape: 'z',
      });
    }
  }
  spawnDanceParticles(cx, cy) {
    const shapes = ['flower', 'leaf'];
    for (let i = 0; i < 20; i++) {
      const shape = shapes[Math.floor(Math.random() * shapes.length)];
      this.particles.push({
        x: cx + (Math.random() - 0.5) * 30, y: cy + (Math.random() - 0.5) * 20,
        vx: (Math.random() - 0.5) * 20, vy: -15 - Math.random() * 30,
        life: 1.0, decay: 0.3 + Math.random() * 0.4,
        size: 2 + Math.floor(Math.random() * 4),
        color: shape === 'flower' ? '#ff9999' : '#88cc66', shape: shape,
      });
    }
  }

  update(dt) {
    for (let i = this.particles.length - 1; i >= 0; i--) {
      const p = this.particles[i];
      p.x += p.vx * dt; p.y += p.vy * dt;
      p.life -= p.decay * dt; p.vy += 10 * dt;
      if (p.life <= 0) this.particles.splice(i, 1);
    }
  }

  render(ctx) {
    ctx.imageSmoothingEnabled = false;
    for (const p of this.particles) {
      ctx.globalAlpha = Math.max(0, p.life);
      ctx.fillStyle = p.color;
      if (p.shape === 'star') {
        ctx.fillRect(p.x - 1, p.y - p.size, 2, p.size * 2);
        ctx.fillRect(p.x - p.size, p.y - 1, p.size * 2, 2);
      } else if (p.shape === 'flower') {
        ctx.fillRect(p.x - 1, p.y - 1, 2, 2);
        ctx.fillRect(p.x - p.size, p.y - 1, p.size * 2 + 2, 2);
        ctx.fillRect(p.x - 1, p.y - p.size, 2, p.size * 2 + 2);
      } else if (p.shape === 'leaf') {
        ctx.fillRect(p.x, p.y, p.size, p.size);
        ctx.fillRect(p.x + 1, p.y - 1, p.size - 1, p.size);
      } else if (p.shape === 'heart') {
        const s = p.size, h = Math.floor(s / 2);
        ctx.fillRect(p.x - s, p.y - h, s, s);
        ctx.fillRect(p.x + 1, p.y - h, s, s);
        ctx.fillRect(p.x - s + 1, p.y, s * 2 - 1, s);
        ctx.fillRect(p.x - h, p.y + s - 1, s * 2, h);
      } else if (p.shape === 'note') {
        const s = p.size;
        ctx.fillRect(p.x, p.y, s + 1, s + 1);
        ctx.fillRect(p.x + s + 1, p.y - s * 2, 1, s * 3);
        ctx.fillRect(p.x + s, p.y - s, s, 1);
      } else if (p.shape === 'z') {
        const s = p.size;
        ctx.fillRect(p.x, p.y - s, s * 3, 1);
        ctx.fillRect(p.x + s, p.y, s * 2, 1);
        ctx.fillRect(p.x, p.y + s, s * 3, 1);
      } else {
        ctx.fillRect(p.x, p.y, p.size, p.size);
      }
    }
    ctx.globalAlpha = 1.0;
  }

  clear() { this.particles = []; }
}
```

---

## sound.js

```javascript
// sound.js — Web Audio API wind chime generator

let audioCtx = null;

function getAudioContext() {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
  return audioCtx;
}

function playWindChime(durationMs) {
  const ctx = getAudioContext();
  if (ctx.state === 'suspended') ctx.resume();
  const notes = [523, 587, 659, 784, 880, 1047, 1175, 1319, 1568];
  const chimeCount = 8;
  const startTime = ctx.currentTime;

  for (let i = 0; i < chimeCount; i++) {
    const delay = Math.random() * (durationMs / 1000) * 0.7;
    const freq = notes[Math.floor(Math.random() * notes.length)];
    const volume = 0.15 + Math.random() * 0.15;
    const dur = 0.8 + Math.random() * 1.5;
    playChimeNote(ctx, freq, volume, dur, startTime + delay);
  }

  // AudioContext stays alive — Web Audio API mixes automatically with other apps
}

function playChimeNote(ctx, frequency, volume, duration, startTime) {
  const osc = ctx.createOscillator();
  const gain = ctx.createGain();
  osc.type = 'sine';
  osc.frequency.value = frequency;
  gain.gain.setValueAtTime(0, startTime);
  gain.gain.linearRampToValueAtTime(volume, startTime + 0.02);
  gain.gain.exponentialRampToValueAtTime(0.001, startTime + duration);
  osc.connect(gain); gain.connect(ctx.destination);
  osc.start(startTime); osc.stop(startTime + duration);

  const osc2 = ctx.createOscillator();
  const gain2 = ctx.createGain();
  osc2.type = 'sine';
  osc2.frequency.value = frequency * 2.01;
  gain2.gain.setValueAtTime(0, startTime);
  gain2.gain.linearRampToValueAtTime(volume * 0.3, startTime + 0.03);
  gain2.gain.exponentialRampToValueAtTime(0.001, startTime + duration * 0.7);
  osc2.connect(gain2); gain2.connect(ctx.destination);
  osc2.start(startTime); osc2.stop(startTime + duration);
}
```

---

## sprites.js

Complete pixel-art definitions for all 6 pets using rectangle-based encoding.
Each pet is a 50x50 coordinate space with color-keyed rectangles.
Animations are generated by `buildPetAnimations()` which applies frame overrides.

```javascript
// sprites.js — Pixel-art sprite definitions for all 6 pets
// Rectangle-based format: { x, y, w, h, c }

function makeAnimFrames(baseRects, overrides) {
  return overrides.map(ov => {
    let frame = baseRects.map(r => {
      const s = ov.shift || { dx: 0, dy: 0 };
      return { x: r.x + (s.dx || 0), y: r.y + (s.dy || 0), w: r.w, h: r.h, c: r.c };
    });
    if (ov.remove) {
      const removeSet = new Set(ov.remove);
      frame = frame.filter((_, i) => !removeSet.has(i));
    }
    if (ov.add) frame = frame.concat(ov.add);
    return frame;
  });
}

// Shared particle rects
const STAR_RECTS = [
  { x: -4, y: -4, w: 2, h: 2, c: '#ffe066' }, { x: 0, y: -6, w: 2, h: 2, c: '#ffe066' },
  { x: 4, y: -4, w: 2, h: 2, c: '#ffe066' }, { x: -2, y: 0, w: 2, h: 2, c: '#ffe066' },
  { x: 2, y: 0, w: 2, h: 2, c: '#ffe066' }, { x: -4, y: 4, w: 2, h: 2, c: '#ffe066' },
  { x: 0, y: 6, w: 2, h: 2, c: '#ffe066' }, { x: 4, y: 4, w: 2, h: 2, c: '#ffe066' },
];
const FLOWER_RECTS = [
  { x: 0, y: -5, w: 3, h: 3, c: '#ff9999' }, { x: -4, y: -2, w: 3, h: 3, c: '#ffaaaa' },
  { x: 4, y: -2, w: 3, h: 3, c: '#ffaaaa' }, { x: -3, y: 3, w: 3, h: 3, c: '#ffbbbb' },
  { x: 3, y: 3, w: 3, h: 3, c: '#ffbbbb' }, { x: 0, y: 0, w: 3, h: 3, c: '#ffdd00' },
];
const LEAF_RECTS = [
  { x: 0, y: -4, w: 2, h: 6, c: '#88cc66' }, { x: 2, y: -3, w: 2, h: 5, c: '#77bb55' },
  { x: -2, y: -1, w: 2, h: 4, c: '#99dd77' },
];

// ===== WHALE =====
const WHALE_BASE = [
  { x: 4, y: 16, w: 40, h: 24, c: 'primary' }, { x: 6, y: 14, w: 36, h: 4, c: 'primary' },
  { x: 2, y: 20, w: 4, h: 16, c: 'primary' }, { x: 42, y: 22, w: 6, h: 14, c: 'primary' },
  { x: 44, y: 18, w: 6, h: 6, c: 'primary' }, { x: 46, y: 12, w: 4, h: 8, c: 'primary' },
  { x: 46, y: 28, w: 4, h: 8, c: 'primary' },
  { x: 8, y: 30, w: 32, h: 8, c: 'secondary' }, { x: 10, y: 28, w: 28, h: 4, c: 'secondary' },
  { x: 8, y: 19, w: 4, h: 4, c: 'outline' }, { x: 9, y: 20, w: 2, h: 2, c: '#ffffff' },
  { x: 10, y: 20, w: 1, h: 1, c: 'outline' },
  { x: 6, y: 26, w: 8, h: 2, c: 'outline' }, { x: 14, y: 22, w: 3, h: 3, c: 'accent' },
  { x: 22, y: 4, w: 4, h: 10, c: '#aaddff' }, { x: 18, y: 6, w: 4, h: 6, c: '#cceeff' },
  { x: 26, y: 6, w: 4, h: 6, c: '#cceeff' }, { x: 20, y: 2, w: 4, h: 4, c: '#ddeeff' },
  { x: 4, y: 24, w: 4, h: 8, c: 'detail' }, { x: 38, y: 20, w: 4, h: 8, c: 'detail' },
];

// ===== CAT — dot eyes + curved tail =====
const CAT_BASE = [
  { x: 12, y: 18, w: 26, h: 24, c: 'primary' }, { x: 14, y: 16, w: 22, h: 4, c: 'primary' },
  { x: 10, y: 20, w: 4, h: 18, c: 'primary' }, { x: 36, y: 20, w: 4, h: 18, c: 'primary' },
  { x: 12, y: 8, w: 8, h: 10, c: 'primary' }, { x: 30, y: 8, w: 8, h: 10, c: 'primary' },
  { x: 14, y: 10, w: 4, h: 6, c: 'secondary' }, { x: 32, y: 10, w: 4, h: 6, c: 'secondary' },
  { x: 18, y: 14, w: 14, h: 8, c: 'secondary' },
  { x: 18, y: 19, w: 5, h: 5, c: '#1a1a1a' }, { x: 27, y: 19, w: 5, h: 5, c: '#1a1a1a' },
  { x: 19, y: 20, w: 1, h: 1, c: '#ffffff' }, { x: 28, y: 20, w: 1, h: 1, c: '#ffffff' },
  { x: 24, y: 26, w: 2, h: 2, c: 'accent' }, { x: 22, y: 28, w: 6, h: 1, c: 'outline' },
  { x: 8, y: 24, w: 6, h: 1, c: 'outline' }, { x: 8, y: 26, w: 6, h: 1, c: 'outline' },
  { x: 36, y: 24, w: 6, h: 1, c: 'outline' }, { x: 36, y: 26, w: 6, h: 1, c: 'outline' },
  { x: 14, y: 40, w: 6, h: 4, c: 'secondary' }, { x: 30, y: 40, w: 6, h: 4, c: 'secondary' },
  { x: 38, y: 30, w: 4, h: 8, c: 'primary' },
  { x: 40, y: 26, w: 4, h: 6, c: 'primary' },
  { x: 42, y: 22, w: 3, h: 6, c: 'primary' },
  { x: 40, y: 28, w: 2, h: 6, c: 'secondary' },
  { x: 18, y: 30, w: 14, h: 10, c: 'secondary' },
];

// ===== DOG — red collar + golden bell =====
const DOG_BASE = [
  { x: 12, y: 20, w: 26, h: 24, c: 'primary' },
  { x: 8, y: 22, w: 6, h: 20, c: 'primary' },
  { x: 36, y: 22, w: 6, h: 20, c: 'primary' },
  { x: 6, y: 8, w: 10, h: 18, c: 'detail' },
  { x: 34, y: 8, w: 10, h: 18, c: 'detail' },
  { x: 14, y: 10, w: 22, h: 14, c: 'primary' },
  { x: 16, y: 12, w: 18, h: 10, c: 'secondary' },
  { x: 18, y: 14, w: 4, h: 4, c: '#1a1a1a' }, { x: 28, y: 14, w: 4, h: 4, c: '#1a1a1a' },
  { x: 19, y: 15, w: 1, h: 1, c: '#ffffff' }, { x: 29, y: 15, w: 1, h: 1, c: '#ffffff' },
  { x: 23, y: 19, w: 4, h: 3, c: 'outline' },
  { x: 22, y: 22, w: 6, h: 2, c: 'outline' },
  { x: 23, y: 24, w: 4, h: 3, c: 'accent' },
  { x: 13, y: 28, w: 24, h: 2, c: '#dd2222' },
  { x: 16, y: 30, w: 18, h: 12, c: 'secondary' },
  { x: 21, y: 30, w: 8, h: 7, c: '#ffcc00' },
  { x: 20, y: 37, w: 10, h: 2, c: '#dd9900' },
  { x: 23, y: 31, w: 2, h: 4, c: '#ffee88' },
  { x: 24, y: 34, w: 2, h: 2, c: '#dd9900' },
  { x: 14, y: 42, w: 6, h: 4, c: 'secondary' }, { x: 30, y: 42, w: 6, h: 4, c: 'secondary' },
  { x: 38, y: 22, w: 5, h: 8, c: 'primary' }, { x: 40, y: 24, w: 3, h: 4, c: 'secondary' },
];

// ===== RABBIT — dot eyes =====
const RABBIT_BASE = [
  { x: 14, y: 22, w: 22, h: 22, c: 'primary' }, { x: 12, y: 24, w: 4, h: 16, c: 'primary' },
  { x: 34, y: 24, w: 4, h: 16, c: 'primary' },
  { x: 16, y: 2, w: 8, h: 22, c: 'primary' }, { x: 26, y: 2, w: 8, h: 22, c: 'primary' },
  { x: 18, y: 4, w: 4, h: 18, c: 'secondary' }, { x: 28, y: 4, w: 4, h: 18, c: 'secondary' },
  { x: 14, y: 16, w: 22, h: 12, c: 'primary' }, { x: 16, y: 18, w: 18, h: 8, c: 'secondary' },
  { x: 18, y: 20, w: 4, h: 4, c: '#1a1a1a' }, { x: 28, y: 20, w: 4, h: 4, c: '#1a1a1a' },
  { x: 19, y: 21, w: 1, h: 1, c: '#ffffff' }, { x: 29, y: 21, w: 1, h: 1, c: '#ffffff' },
  { x: 24, y: 25, w: 2, h: 2, c: 'accent' }, { x: 14, y: 28, w: 4, h: 2, c: 'accent' },
  { x: 32, y: 28, w: 4, h: 2, c: 'accent' },
  { x: 16, y: 42, w: 5, h: 4, c: 'secondary' }, { x: 29, y: 42, w: 5, h: 4, c: 'secondary' },
  { x: 36, y: 32, w: 6, h: 6, c: '#ffffff' }, { x: 18, y: 28, w: 14, h: 12, c: 'secondary' },
];

// ===== HAMSTER — dot eyes =====
const HAMSTER_BASE = [
  { x: 10, y: 14, w: 30, h: 28, c: 'primary' }, { x: 12, y: 12, w: 26, h: 4, c: 'primary' },
  { x: 8, y: 18, w: 4, h: 20, c: 'primary' }, { x: 38, y: 18, w: 4, h: 20, c: 'primary' },
  { x: 12, y: 6, w: 7, h: 7, c: 'primary' }, { x: 31, y: 6, w: 7, h: 7, c: 'primary' },
  { x: 14, y: 8, w: 4, h: 4, c: 'secondary' }, { x: 33, y: 8, w: 4, h: 4, c: 'secondary' },
  { x: 18, y: 14, w: 14, h: 12, c: 'secondary' }, { x: 20, y: 12, w: 10, h: 4, c: 'secondary' },
  { x: 17, y: 18, w: 5, h: 5, c: '#1a1a1a' }, { x: 28, y: 18, w: 5, h: 5, c: '#1a1a1a' },
  { x: 18, y: 19, w: 1, h: 1, c: '#ffffff' }, { x: 29, y: 19, w: 1, h: 1, c: '#ffffff' },
  { x: 24, y: 24, w: 2, h: 2, c: 'accent' }, { x: 22, y: 26, w: 6, h: 1, c: 'outline' },
  { x: 12, y: 24, w: 6, h: 4, c: 'secondary' }, { x: 32, y: 24, w: 6, h: 4, c: 'secondary' },
  { x: 14, y: 40, w: 5, h: 3, c: 'secondary' }, { x: 31, y: 40, w: 5, h: 3, c: 'secondary' },
  { x: 18, y: 28, w: 14, h: 12, c: 'secondary' },
];

// ===== OCTOPUS — red, wavy tentacles, dot mouth =====
const OCTOPUS_BASE = [
  { x: 13, y: 4, w: 24, h: 3, c: 'primary' },
  { x: 10, y: 7, w: 30, h: 16, c: 'primary' },
  { x: 8, y: 9, w: 4, h: 12, c: 'primary' },
  { x: 38, y: 9, w: 4, h: 12, c: 'primary' },
  { x: 16, y: 5, w: 18, h: 4, c: '#ff8888' },
  { x: 16, y: 12, w: 4, h: 4, c: '#1a1a1a' }, { x: 30, y: 12, w: 4, h: 4, c: '#1a1a1a' },
  { x: 17, y: 13, w: 1, h: 1, c: '#ffffff' }, { x: 31, y: 13, w: 1, h: 1, c: '#ffffff' },
  { x: 12, y: 17, w: 4, h: 3, c: '#ffaaaa' }, { x: 34, y: 17, w: 4, h: 3, c: '#ffaaaa' },
  { x: 23, y: 19, w: 4, h: 3, c: '#881111' },
  { x: 15, y: 22, w: 20, h: 4, c: 'primary' },
  // 8 wavy tentacles (each ~5 segmented rects with alternating x)
  { x: 5, y: 26, w: 5, h: 4, c: 'primary' }, { x: 4, y: 30, w: 4, h: 5, c: 'primary' },
  { x: 5, y: 35, w: 5, h: 5, c: 'primary' }, { x: 4, y: 40, w: 4, h: 4, c: 'primary' },
  { x: 5, y: 44, w: 3, h: 2, c: '#ff7777' },
  { x: 10, y: 26, w: 5, h: 4, c: 'primary' }, { x: 11, y: 30, w: 4, h: 5, c: 'primary' },
  { x: 10, y: 35, w: 5, h: 5, c: 'primary' }, { x: 11, y: 40, w: 4, h: 4, c: 'primary' },
  { x: 10, y: 44, w: 3, h: 2, c: '#ff7777' },
  { x: 15, y: 27, w: 4, h: 4, c: 'primary' }, { x: 14, y: 31, w: 5, h: 4, c: 'primary' },
  { x: 15, y: 35, w: 4, h: 5, c: 'primary' }, { x: 14, y: 40, w: 4, h: 4, c: 'primary' },
  { x: 15, y: 44, w: 3, h: 2, c: '#ff7777' },
  { x: 20, y: 27, w: 4, h: 4, c: 'primary' }, { x: 21, y: 31, w: 4, h: 4, c: 'primary' },
  { x: 20, y: 35, w: 5, h: 5, c: 'primary' }, { x: 21, y: 40, w: 3, h: 4, c: 'primary' },
  { x: 20, y: 44, w: 3, h: 2, c: '#ff7777' },
  { x: 26, y: 27, w: 4, h: 4, c: 'primary' }, { x: 25, y: 31, w: 4, h: 4, c: 'primary' },
  { x: 26, y: 35, w: 4, h: 5, c: 'primary' }, { x: 25, y: 40, w: 4, h: 4, c: 'primary' },
  { x: 26, y: 44, w: 3, h: 2, c: '#ff7777' },
  { x: 31, y: 27, w: 4, h: 4, c: 'primary' }, { x: 32, y: 31, w: 4, h: 4, c: 'primary' },
  { x: 31, y: 35, w: 5, h: 5, c: 'primary' }, { x: 32, y: 40, w: 4, h: 4, c: 'primary' },
  { x: 31, y: 44, w: 3, h: 2, c: '#ff7777' },
  { x: 36, y: 26, w: 5, h: 4, c: 'primary' }, { x: 35, y: 30, w: 4, h: 5, c: 'primary' },
  { x: 36, y: 35, w: 5, h: 5, c: 'primary' }, { x: 35, y: 40, w: 4, h: 4, c: 'primary' },
  { x: 36, y: 44, w: 3, h: 2, c: '#ff7777' },
  { x: 41, y: 26, w: 5, h: 4, c: 'primary' }, { x: 42, y: 30, w: 4, h: 5, c: 'primary' },
  { x: 41, y: 35, w: 5, h: 5, c: 'primary' }, { x: 42, y: 40, w: 4, h: 4, c: 'primary' },
  { x: 41, y: 44, w: 3, h: 2, c: '#ff7777' },
];

// Color maps
const WHALE_COLORS = { primary: '#6699cc', secondary: '#cce0f0', accent: '#ffaaaa', outline: '#334455', detail: '#5588bb' };
const CAT_COLORS = { primary: '#ffcc99', secondary: '#fff0e0', accent: '#ff8888', outline: '#554433', detail: '#ffaa77' };
const DOG_COLORS = { primary: '#deb887', secondary: '#fff8f0', accent: '#ff6666', outline: '#443322', detail: '#c4956a' };
const RABBIT_COLORS = { primary: '#f5f0e8', secondary: '#ffffff', accent: '#ffaaaa', outline: '#998877', detail: '#ddd8cc' };
const HAMSTER_COLORS = { primary: '#e8c882', secondary: '#fffdf0', accent: '#ff9999', outline: '#665533', detail: '#d4a854' };
const OCTOPUS_COLORS = { primary: '#ff5555', secondary: '#ffcccc', accent: '#ffaaaa', outline: '#331111', detail: '#ff7777' };

function buildPetAnimations(baseRects) {
  return {
    idle: makeAnimFrames(baseRects, [{}, { shift: { dx: 0, dy: 1 } }]),
    jump: makeAnimFrames(baseRects, [
      { shift: { dx: 0, dy: 0 } },
      { shift: { dx: 0, dy: -6 } },
      { shift: { dx: 0, dy: -10 }, add: STAR_RECTS.map(r => ({ ...r, x: r.x + 25, y: r.y + 20 })) },
      { shift: { dx: 0, dy: -4 }, add: STAR_RECTS.map(r => ({ ...r, x: r.x + 25, y: r.y + 15 })) },
    ]),
    drink: makeAnimFrames(baseRects, [
      {},
      { shift: { dx: 0, dy: 1 }, add: [{ x: 12, y: 2, w: 8, h: 10, c: '#88ccff' }, { x: 14, y: 4, w: 4, h: 6, c: '#aaddff' }, { x: 28, y: 30, w: 6, h: 4, c: '#88ccff' }] },
      { shift: { dx: 0, dy: 0 }, add: [{ x: 10, y: 6, w: 8, h: 10, c: '#88ccff' }, { x: 12, y: 8, w: 4, h: 4, c: '#aaddff' }, { x: 28, y: 28, w: 6, h: 4, c: '#88ccff' }] },
    ]),
    stretch: makeAnimFrames(baseRects, [
      {},
      { shift: { dx: 0, dy: -2 }, add: [{ x: 8, y: 4, w: 4, h: 6, c: 'primary' }, { x: 38, y: 4, w: 4, h: 6, c: 'primary' }] },
      { shift: { dx: 0, dy: -3 }, add: [{ x: 8, y: 2, w: 4, h: 8, c: 'primary' }, { x: 38, y: 2, w: 4, h: 8, c: 'primary' }, { x: 20, y: 0, w: 4, h: 4, c: '#ffdd88' }, { x: 30, y: 2, w: 2, h: 2, c: '#ffdd88' }] },
    ]),
    bath: makeAnimFrames(baseRects, [
      {},
      { shift: { dx: 0, dy: 0 }, add: [{ x: 18, y: 2, w: 4, h: 4, c: '#88ccff' }, { x: 30, y: 4, w: 3, h: 3, c: '#88ccff' }] },
      { shift: { dx: 1, dy: 0 }, add: [{ x: 14, y: 0, w: 4, h: 4, c: '#88ccff' }, { x: 28, y: 2, w: 3, h: 3, c: '#88ccff' }, { x: 22, y: 6, w: 4, h: 4, c: '#aaddff' }, { x: 34, y: 0, w: 3, h: 3, c: '#88ccff' }] },
    ]),
    eatFruit: makeAnimFrames(baseRects, [
      {},
      { add: [{ x: 34, y: 26, w: 8, h: 8, c: '#ff6666' }, { x: 36, y: 24, w: 4, h: 2, c: '#66aa44' }, { x: 22, y: 32, w: 6, h: 4, c: 'primary' }] },
      { add: [{ x: 36, y: 28, w: 6, h: 6, c: '#ff6666' }, { x: 36, y: 24, w: 4, h: 2, c: '#66aa44' }, { x: 20, y: 34, w: 6, h: 4, c: 'primary' }, { x: 14, y: 2, w: 3, h: 3, c: '#ffdd88' }] },
    ]),
    eatCake: makeAnimFrames(baseRects, [
      {},
      { add: [{ x: 32, y: 24, w: 10, h: 8, c: '#ffcc88' }, { x: 34, y: 20, w: 6, h: 6, c: '#ffaaaa' }, { x: 36, y: 18, w: 2, h: 4, c: '#ff4444' }] },
      { add: [{ x: 34, y: 26, w: 8, h: 6, c: '#ffcc88' }, { x: 36, y: 22, w: 4, h: 6, c: '#ffaaaa' }, { x: 38, y: 20, w: 2, h: 4, c: '#ff4444' }, { x: 14, y: 0, w: 3, h: 3, c: '#ffdd88' }, { x: 28, y: 2, w: 3, h: 3, c: '#ffdd88' }] },
    ]),
    massage: makeAnimFrames(baseRects, [
      {}, { shift: { dx: -2, dy: 0 } }, { shift: { dx: 2, dy: 0 } }, { shift: { dx: -1, dy: 1 } },
    ]),
    dance: makeAnimFrames(baseRects, [
      { shift: { dx: 0, dy: 0 } },
      { shift: { dx: -3, dy: -2 }, add: FLOWER_RECTS.map(r => ({ ...r, x: r.x + 10, y: r.y + 40 })) },
      { shift: { dx: 3, dy: -1 }, add: LEAF_RECTS.map(r => ({ ...r, x: r.x + 40, y: r.y + 38 })) },
      { shift: { dx: -2, dy: -2 }, add: FLOWER_RECTS.map(r => ({ ...r, x: r.x + 40, y: r.y + 40 })) },
      { shift: { dx: 2, dy: -1 }, add: LEAF_RECTS.map(r => ({ ...r, x: r.x + 10, y: r.y + 38 })) },
      { shift: { dx: 0, dy: 0 } },
    ]),
  };
}

const PETS = {
  whale: { name: '鲸鱼', nameEn: 'whale', colors: WHALE_COLORS, baseRects: WHALE_BASE, animations: buildPetAnimations(WHALE_BASE) },
  cat: { name: '猫咪', nameEn: 'cat', colors: CAT_COLORS, baseRects: CAT_BASE, animations: buildPetAnimations(CAT_BASE) },
  dog: { name: '小狗', nameEn: 'dog', colors: DOG_COLORS, baseRects: DOG_BASE, animations: buildPetAnimations(DOG_BASE) },
  rabbit: { name: '兔子', nameEn: 'rabbit', colors: RABBIT_COLORS, baseRects: RABBIT_BASE, animations: buildPetAnimations(RABBIT_BASE) },
  hamster: { name: '仓鼠', nameEn: 'hamster', colors: HAMSTER_COLORS, baseRects: HAMSTER_BASE, animations: buildPetAnimations(HAMSTER_BASE) },
  octopus: { name: '章鱼', nameEn: 'octopus', colors: OCTOPUS_COLORS, baseRects: OCTOPUS_BASE, animations: buildPetAnimations(OCTOPUS_BASE) },
};
```
