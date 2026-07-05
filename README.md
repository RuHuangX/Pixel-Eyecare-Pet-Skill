# 🐋 Pixel Eye Care Pet — 像素护眼桌面宠物
 A Claude Code skill that generates a complete Electron desktop pet app. Features 6 cute pixel-art companions, 20-20-20 eye care rule reminders, smart fullscreen detection (auto-hide during gaming), Game &amp; Companion modes, Web Audio wind chimes, black hole hide/show animations, and idle leisure animations. 

> 一个 Claude Code Skill，以 20-20-20 法则创建用于生成完整的像素风护眼桌面宠物应用。

调用此 Skill 后，Claude 会自动生成一个可直接运行的 Electron 桌面应用——一只可拖拽的像素宠物常驻桌面，静默计时 20 分钟，温柔提醒你休息眼睛。

<p align="center">
  <b>🐋 鲸鱼 &nbsp;|&nbsp; 🐙 章鱼 &nbsp;|&nbsp; 🐰 兔子 &nbsp;|&nbsp; 🐱 猫咪 &nbsp;|&nbsp; 🐶 小狗 &nbsp;|&nbsp; 🐹 仓鼠</b>
</p>
![Uploading 6af8e38812a35f8999549978587a6f31.gif…]()

---

## ✨ 功能介绍

### 🎨 个性化选择

六种像素宠物伙伴，随心切换，共包含：

- **鲸鱼** 🐋 
- **章鱼** 🐙 
- **兔子** 🐰 
- **猫咪** 🐱 
- **小狗** 🐶 
- **仓鼠** 🐹 

具体形象以应用内为主，所有宠物均配有完整动画集。

### ⏰ 20-20-20 护眼提醒

- 静默 20 分钟倒计时，无可见计时器
- 提醒触发：宠物特效 + 屏幕中央弹出像素对话框
- **风铃音效**：Web Audio API 合成，无需任何音频文件
- 点击"确认"关闭弹窗，计时器立即重置

### 🎮 游戏模式

- 右键宠物开启
- 护眼提醒弹窗**不抢占全屏画面**（避免游戏闪退）
- 风铃声**每 7 秒循环播放**，直到手动确认停止
- 开启游戏模式时自动重新开始 20 分钟计时

### 💤 陪伴模式

- 右键宠物开启
- **不触发护眼提醒**
- 适合同时运行多只宠物，只需一只有提醒

### 🔍 智能全屏检测

- 普通桌面使用：宠物**始终置顶显示**
- 自动**检测全屏程序**（游戏、视频、直播）
- 检测到全屏 → **自动隐藏宠物**
- 退出全屏 → **自动恢复显示**
- 后台计时和状态全程保持

### 🕳️ 隐藏与重新显示

- 右键 → 隐藏：宠物消失，后台运行
- 系统托盘 → 重新显示：宠物可视

### 🔄 重新开始（爱心跳跃）

- 右键或托盘 → 重新开始：宠物以默认初始模式开始计时
- 所有特殊模式退出，恢复初始状态

### 🎪 彩蛋

每隔约 5 分钟（±30 秒随机浮动），宠物会自动表演一段**5 秒无声动画**


### 🖱️ 桌面集成

- **可拖拽** — 按住宠物拖到桌面任意位置
- **位置记忆** — 关闭后重新打开，宠物在你上次放的位置
- **操作界面** — 右键小宠物进行菜单操作
- **桌面快捷方式** — 生成像素眼睛 `.ico` 图标
- **56×56 像素窗口** — 极小占地面积，透明边缘不阻挡点击

---

## 🚀 使用方式

### 环境要求
- **Claude Code**（或任意支持 Skill 的 Claude 环境）
- Skill 需放置在 `.claude/skills/` 目录下
- 或自行配置为可兼容其他agent的skill文件

### 安装 Skill
```bash
# 克隆到 skills 目录
git clone <仓库地址> ~/.claude/skills/pixel-eye-care-pet

# 或手动将 pixel-eye-care-pet 文件夹复制到 .claude/skills/ 下
```

### 生成桌面应用
在 Claude Code 中直接说：

> "帮我创建一个像素护眼桌面宠物"  
> "做一个20-20-20护眼小应用"  
> "我想要一只桌面猫咪提醒我休息眼睛"

Claude 会自动调用 Skill，生成完整的 Electron 项目。

### 运行生成的应用
```bash
cd pixel-eye-care-pet
npm install
npm start
```
---

## 📁 Skill 目录结构

```
pixel-eye-care-pet/
├── SKILL.md                         # Skill 主指令文件
├── README.md                        # 本文件
└── references/
    ├── electron-core.md             # main.js + preload.js + 子窗口页面
    ├── renderer-core.md             # index.html + style.css + app.js
    └── pixel-engine.md              # 精灵图 + 粒子系统 + 音效 + 渲染器
```

## 🛠️ 生成应用的技术栈

| 层级 | 技术 |
|---|---|
| 桌面框架 | **Electron 28** |
| 图形渲染 | **Canvas 2D**（像素画） |
| 音频 | **Web Audio API**（合成音效） |
| 打包 | **electron-builder**（可选） |
| 外部依赖 | **零**（仅需 Electron） |



## 📄 许可协议

GPL-3.0

---

眼睛是欣赏世界的窗口，工作繁忙，别忘了爱护双眼，欢迎共创 ❤️
