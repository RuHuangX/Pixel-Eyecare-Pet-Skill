# Electron Core — main.js + preload.js + sub-windows

Final v3 architecture: main window is always 56x56. Right-click menu, pet
selection, and reminders are all separate BrowserWindows — zero air wall.

The reference includes: main.js, preload.js, reminder.html, menu.html, selection.html.

---

## main.js

```javascript
const { app, BrowserWindow, Tray, Menu, ipcMain, nativeImage, screen } = require('electron');
const path = require('path');
const fs = require('fs');

let mainWindow = null;
let tray = null;
let reminderWindow = null;
let selectionWindow = null;
let menuWindow = null;
let isQuitting = false;
let petPosition = null;
let petType = 'whale';

const CONFIG_PATH = path.join(app.getPath('userData'), 'pet-config.json');

function loadConfig() {
  try {
    if (fs.existsSync(CONFIG_PATH)) {
      const data = JSON.parse(fs.readFileSync(CONFIG_PATH, 'utf8'));
      petPosition = data.position || null;
      petType = data.petType || 'whale';
      return;
    }
  } catch (e) {}
  petPosition = null; petType = 'whale';
}

function saveConfig() {
  try {
    fs.writeFileSync(CONFIG_PATH, JSON.stringify({ position: petPosition, petType }, null, 2), 'utf8');
  } catch (e) {}
}

function createMainWindow() {
  const { width, height } = screen.getPrimaryDisplay().workAreaSize;
  const winOptions = {
    width: 56, height: 56,
    transparent: true, frame: false,
    alwaysOnTop: true, resizable: false,
    skipTaskbar: true, type: 'panel',
    webPreferences: {
      preload: path.join(__dirname, 'preload.js'),
      contextIsolation: true, nodeIntegration: false,
    },
  };
  let px = petPosition ? petPosition.x : width - 100;
  let py = petPosition ? petPosition.y : height - 100;
  if (px < -50 || px > width - 20 || py < -50 || py > height - 20) {
    px = width - 100; py = height - 100;
  }
  winOptions.x = px; winOptions.y = py;

  mainWindow = new BrowserWindow(winOptions);
  mainWindow.loadFile(path.join(__dirname, 'renderer', 'index.html'));
  mainWindow.show();
  // Smart fullscreen detection: hide pet when fullscreen app is active
  let fullscreenHidden = false;
  setInterval(() => {
    if (!mainWindow || mainWindow.isDestroyed()) return;
    const disp = screen.getPrimaryDisplay();
    const { workArea, bounds } = disp;
    const likelyFullscreen = (bounds.height - workArea.height < 40) && (bounds.width - workArea.width < 40);
    if (likelyFullscreen && !fullscreenHidden) {
      if (mainWindow.isVisible()) mainWindow.hide();
      if (reminderWindow) { reminderWindow.close(); reminderWindow = null; }
      fullscreenHidden = true;
    } else if (!likelyFullscreen && fullscreenHidden) {
      if (!mainWindow.isVisible()) { mainWindow.show(); mainWindow.focus(); }
      fullscreenHidden = false;
    }
  }, 3000);

  mainWindow.on('close', (event) => {
    if (!isQuitting) { event.preventDefault(); mainWindow.hide(); }
  });
  mainWindow.on('move', () => {
    if (mainWindow) { const [x, y] = mainWindow.getPosition(); petPosition = { x, y }; }
  });
}

function showReminderDialog(gameMode) {
  if (reminderWindow) { reminderWindow.focus(); return; }
  const { width, height } = screen.getPrimaryDisplay().workAreaSize;
  reminderWindow = new BrowserWindow({
    width: 300, height: 220,
    x: Math.round((width - 300) / 2), y: Math.round((height - 220) / 2),
    frame: false, transparent: true,
    alwaysOnTop: !gameMode, resizable: false,
    skipTaskbar: true, parent: mainWindow, modal: false,
    webPreferences: { preload: path.join(__dirname, 'preload.js'), contextIsolation: true, nodeIntegration: false },
  });
  reminderWindow.loadFile(path.join(__dirname, 'renderer', 'reminder.html'));
  reminderWindow.on('closed', () => { reminderWindow = null; });
}

function createPixelEyeIcon() {
  const size = 16;
  const buf = Buffer.alloc(size * size * 4);
  for (let y = 0; y < size; y++) {
    for (let x = 0; x < size; x++) {
      const dx = x - 7, dy = y - 7;
      const de = Math.sqrt((x-7)*(x-7)*0.7 + (y-7)*(y-7));
      const di = Math.sqrt(dx*dx + dy*dy);
      const ds = Math.sqrt((x-5)*(x-5) + (y-5)*(y-5));
      const i = (y * size + x) * 4;
      if (ds <= 1.2)      { buf[i]=255; buf[i+1]=255; buf[i+2]=255; buf[i+3]=255; }
      else if (di <= 1.8) { buf[i]=30;  buf[i+1]=25;  buf[i+2]=20;  buf[i+3]=255; }
      else if (di <= 3.5) { buf[i]=139; buf[i+1]=90;  buf[i+2]=43;  buf[i+3]=255; }
      else if (de <= 5.5) { buf[i]=245; buf[i+1]=240; buf[i+2]=235; buf[i+3]=255; }
    }
  }
  return buf;
}

function createTray() {
  const iconPath = path.join(__dirname, 'icon.png');
  let trayIcon;
  if (fs.existsSync(iconPath)) {
    trayIcon = nativeImage.createFromPath(iconPath).resize({ width: 16, height: 16 });
  } else {
    trayIcon = nativeImage.createFromBuffer(createPixelEyeIcon(), { width: 16, height: 16, scaleFactor: 1.0 });
  }
  tray = new Tray(trayIcon);
  tray.setToolTip('Pixel Eye Care Pet - 20-20-20');
  tray.setContextMenu(Menu.buildFromTemplate([
    { label: '重新显示', click: () => {
      if (mainWindow) {
        const { width, height } = screen.getPrimaryDisplay().workAreaSize;
        const [x, y] = mainWindow.getPosition();
        if (x < -50 || x > width - 20 || y < -50 || y > height - 20) {
          mainWindow.setPosition(width - 100, height - 100);
        }
        mainWindow.show(); mainWindow.focus();
        mainWindow.webContents.send('show-dance');
      }
    }},
    { label: '重新开始', click: () => {
      if (mainWindow) { mainWindow.show(); mainWindow.webContents.send('reset-timer'); }
    }},
    { type: 'separator' },
    { label: '退出', click: () => {
      isQuitting = true; saveConfig();
      if (reminderWindow) { reminderWindow.close(); reminderWindow = null; }
      app.quit();
    }},
  ]));
  tray.on('double-click', () => { if (mainWindow) { mainWindow.show(); mainWindow.focus(); } });
}

function ensureIcon() {
  const iconPath = path.join(__dirname, 'icon.png');
  if (!fs.existsSync(iconPath)) {
    try {
      const img = nativeImage.createFromBuffer(createPixelEyeIcon(), { width: 16, height: 16, scaleFactor: 1.0 });
      fs.writeFileSync(iconPath, img.toPNG());
    } catch (e) {}
  }
}

function setupIPC() {
  ipcMain.handle('save-position', (_e, { x, y }) => { petPosition = { x, y }; saveConfig(); return true; });
  ipcMain.handle('get-position', () => petPosition);
  ipcMain.handle('save-pet-type', (_e, type) => { petType = type; saveConfig(); return true; });
  ipcMain.handle('get-pet-type', () => petType);
  ipcMain.handle('show-reminder', (_e, gameMode) => { showReminderDialog(!!gameMode); return true; });
  ipcMain.handle('close-reminder', () => {
    if (reminderWindow) { reminderWindow.close(); reminderWindow = null; }
    if (mainWindow) mainWindow.webContents.send('reminder-dismissed');
    return true;
  });
  ipcMain.handle('quit-app', () => {
    isQuitting = true; saveConfig();
    if (reminderWindow) { reminderWindow.close(); reminderWindow = null; }
    app.quit(); return true;
  });
  ipcMain.handle('hide-window', () => {
    if (mainWindow) mainWindow.hide();
    if (reminderWindow) { reminderWindow.close(); reminderWindow = null; }
    return true;
  });
  ipcMain.handle('show-window', () => {
    if (mainWindow) {
      const { width, height } = screen.getPrimaryDisplay().workAreaSize;
      const [x, y] = mainWindow.getPosition();
      if (x < -50 || x > width - 20 || y < -50 || y > height - 20) {
        mainWindow.setPosition(width - 100, height - 100);
      }
      mainWindow.show(); mainWindow.focus();
      mainWindow.webContents.send('show-dance');
    }
    return true;
  });
  ipcMain.handle('get-screen-bounds', () => {
    const { width, height } = screen.getPrimaryDisplay().workAreaSize;
    return { width, height };
  });
  ipcMain.on('move-window', (_e, { dx, dy }) => {
    if (mainWindow) { const [x, y] = mainWindow.getPosition(); mainWindow.setPosition(x + dx, y + dy); }
  });

  // Pixel context menu — separate window, no air wall
  ipcMain.handle('show-context-menu', () => {
    if (menuWindow) { menuWindow.close(); }
    const [px, py] = mainWindow ? mainWindow.getPosition() : [100, 100];
    menuWindow = new BrowserWindow({
      width: 130, height: 155, x: px + 56, y: py - 5,
      frame: false, transparent: true, alwaysOnTop: true, resizable: false, skipTaskbar: true,
      webPreferences: { preload: path.join(__dirname, 'preload.js'), contextIsolation: true, nodeIntegration: false },
    });
    menuWindow.loadFile(path.join(__dirname, 'renderer', 'menu.html'));
    menuWindow.on('closed', () => { menuWindow = null; });
    menuWindow.on('blur', () => { if (menuWindow) { menuWindow.close(); menuWindow = null; } });
    return true;
  });
  ipcMain.handle('menu-action', (_e, action) => {
    if (menuWindow) { menuWindow.close(); menuWindow = null; }
    if (mainWindow) {
      if (action === 'reset') mainWindow.webContents.send('reset-timer');
      if (action === 'switch') mainWindow.webContents.send('switch-pet');
      if (action === 'hide') mainWindow.webContents.send('hide-pet');
      if (action === 'game') mainWindow.webContents.send('toggle-game');
      if (action === 'companion') mainWindow.webContents.send('toggle-companion');
    }
    return true;
  });
  ipcMain.handle('close-menu', () => { if (menuWindow) { menuWindow.close(); menuWindow = null; } return true; });

  // Selection panel — separate window
  ipcMain.handle('show-selection-panel', () => {
    if (selectionWindow) { selectionWindow.focus(); return true; }
    const { width, height } = screen.getPrimaryDisplay().workAreaSize;
    selectionWindow = new BrowserWindow({
      width: 240, height: 280,
      x: Math.round((width - 240) / 2), y: Math.round((height - 280) / 2),
      frame: false, transparent: true, alwaysOnTop: true, resizable: false,
      skipTaskbar: true, parent: mainWindow,
      webPreferences: { preload: path.join(__dirname, 'preload.js'), contextIsolation: true, nodeIntegration: false },
    });
    selectionWindow.loadFile(path.join(__dirname, 'renderer', 'selection.html'));
    selectionWindow.on('closed', () => { selectionWindow = null; });
    return true;
  });
  ipcMain.handle('close-selection', () => {
    if (selectionWindow) { selectionWindow.close(); selectionWindow = null; loadConfig(); if (mainWindow) mainWindow.webContents.send('pet-type-changed', petType); }
    return true;
  });
}

app.whenReady().then(() => { loadConfig(); setupIPC(); createMainWindow(); createTray(); ensureIcon(); });
app.on('before-quit', () => { isQuitting = true; saveConfig(); });
app.on('activate', () => { if (mainWindow) mainWindow.show(); else createMainWindow(); });
```

---

## preload.js

```javascript
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('electronAPI', {
  savePosition: (x, y) => ipcRenderer.invoke('save-position', { x, y }),
  getPosition: () => ipcRenderer.invoke('get-position'),
  savePetType: (type) => ipcRenderer.invoke('save-pet-type', type),
  getPetType: () => ipcRenderer.invoke('get-pet-type'),
  showReminder: (gameMode) => ipcRenderer.invoke('show-reminder', gameMode),
  closeReminder: () => ipcRenderer.invoke('close-reminder'),
  quitApp: () => ipcRenderer.invoke('quit-app'),
  moveWindow: (dx, dy) => ipcRenderer.send('move-window', { dx, dy }),
  showContextMenu: () => ipcRenderer.invoke('show-context-menu'),
  showSelectionPanel: () => ipcRenderer.invoke('show-selection-panel'),
  closeSelection: () => ipcRenderer.invoke('close-selection'),
  menuAction: (action) => ipcRenderer.invoke('menu-action', action),
  closeMenu: () => ipcRenderer.invoke('close-menu'),
  hideWindow: () => ipcRenderer.invoke('hide-window'),
  showWindow: () => ipcRenderer.invoke('show-window'),
  getScreenBounds: () => ipcRenderer.invoke('get-screen-bounds'),
  onShowDance: (cb) => { ipcRenderer.on('show-dance', () => cb()); return () => ipcRenderer.removeAllListeners('show-dance'); },
  onResetTimer: (cb) => { ipcRenderer.on('reset-timer', () => cb()); return () => ipcRenderer.removeAllListeners('reset-timer'); },
  onReminderDismissed: (cb) => { ipcRenderer.on('reminder-dismissed', () => cb()); return () => ipcRenderer.removeAllListeners('reminder-dismissed'); },
  onSwitchPet: (cb) => { ipcRenderer.on('switch-pet', () => cb()); return () => ipcRenderer.removeAllListeners('switch-pet'); },
  onHidePet: (cb) => { ipcRenderer.on('hide-pet', () => cb()); return () => ipcRenderer.removeAllListeners('hide-pet'); },
  onToggleGame: (cb) => { ipcRenderer.on('toggle-game', () => cb()); return () => ipcRenderer.removeAllListeners('toggle-game'); },
  onToggleCompanion: (cb) => { ipcRenderer.on('toggle-companion', () => cb()); return () => ipcRenderer.removeAllListeners('toggle-companion'); },
  onPetTypeChanged: (cb) => { ipcRenderer.on('pet-type-changed', (_e, type) => cb(type)); return () => ipcRenderer.removeAllListeners('pet-type-changed'); },
});
```

---

## menu.html — Pixel context menu (separate window)

```html
<!DOCTYPE html><html lang="zh-CN"><head><meta charset="UTF-8"><style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: transparent; overflow: hidden; font-family: 'Microsoft YaHei','PingFang SC',monospace; -webkit-app-region: no-drag; user-select: none; }
.menu { background: #fff8f0; border: 3px solid #5a3a2a; padding: 2px 0; box-shadow: 3px 0 0 #5a3a2a,-3px 0 0 #5a3a2a,0 3px 0 #5a3a2a,0 -3px 0 #5a3a2a,0 0 0 3px #c8a882; }
.item { padding: 6px 14px; cursor: pointer; color: #5a3a2a; font-size: 11px; white-space: nowrap; }
.item:hover { background: #ffe0cc; color: #3a1a0a; }
</style></head><body>
<div class="menu">
  <div class="item" data-action="reset">🔄 重新开始</div>
  <div class="item" data-action="switch">🎨 切换形象</div>
  <div class="item" data-action="hide">👻 隐藏</div>
  <div class="item" data-action="game">🎮 游戏模式</div>
  <div class="item" data-action="companion">💤 陪伴模式</div>
</div>
<script>
document.querySelector('.menu').addEventListener('click', (e) => {
  const item = e.target.closest('.item'); if (!item) return;
  window.electronAPI.menuAction(item.dataset.action);
});
document.body.addEventListener('click', (e) => {
  if (!e.target.closest('.item')) window.electronAPI.closeMenu();
});
</script>
</body></html>
```

---

## selection.html — Pet selection panel (separate window)

```html
<!DOCTYPE html><html lang="zh-CN"><head><meta charset="UTF-8"><style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { background: transparent; display: flex; justify-content: center; align-items: center; height: 100vh; overflow: hidden; font-family: 'Microsoft YaHei','PingFang SC',monospace; -webkit-app-region: no-drag; user-select: none; }
.panel { background: #fff8f0; border: 4px solid #5a3a2a; padding: 12px; text-align: center; box-shadow: 4px 0 0 #5a3a2a,-4px 0 0 #5a3a2a,0 4px 0 #5a3a2a,0 -4px 0 #5a3a2a,0 0 0 4px #c8a882; }
.title { font-size: 14px; color: #fff; background: #5a3a2a; padding: 6px 16px; margin-bottom: 10px; border: 2px solid #c8a882; }
.grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 6px; }
.card { display: flex; flex-direction: column; align-items: center; padding: 6px; cursor: pointer; border: 2px solid transparent; }
.card:hover { border-color: #ff9966; background: #fff0e0; }
canvas { width: 50px; height: 50px; image-rendering: pixelated; margin-bottom: 2px; }
span { font-size: 11px; color: #5a3a2a; }
</style></head><body>
<div class="panel">
  <div class="title">选择你的护眼小伙伴吧！</div>
  <div class="grid" id="grid"></div>
</div>
<script src="sprites.js"></script>
<script src="canvas.js"></script>
<script>
const grid = document.getElementById('grid');
const pets = ['whale','octopus','rabbit','cat','dog','hamster'];
const emoji = { whale:'🐋', octopus:'🐙', rabbit:'🐰', cat:'🐱', dog:'🐶', hamster:'🐹' };
pets.forEach(key => {
  const card = document.createElement('div'); card.className = 'card';
  const cvs = document.createElement('canvas'); cvs.width = 50; cvs.height = 50;
  const ctx = cvs.getContext('2d'); ctx.imageSmoothingEnabled = false;
  const config = PETS[key];
  if (config) renderSprite(ctx, config.animations.idle[0], config.colors, 0, 0, 1);
  const label = document.createElement('span');
  label.textContent = emoji[key] + ' ' + config.name;
  card.appendChild(cvs); card.appendChild(label);
  card.addEventListener('click', async () => { await window.electronAPI.savePetType(key); window.electronAPI.closeSelection(); });
  grid.appendChild(card);
});
</script>
</body></html>
```

---

## reminder.html

(Unchanged from v2 — pixel-styled centered dialog, see previous version)
