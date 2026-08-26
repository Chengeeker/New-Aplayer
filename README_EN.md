# New-Aplayer (iOS Liquid Glass Edition)

<div align="center">

[![Hexo](https://img.shields.io/badge/Hexo-5.0+-blue.svg)](https://hexo.io/)
[![APlayer](https://img.shields.io/badge/APlayer-1.10.1-red.svg)](https://aplayer.js.org/)
[![MetingJS](https://img.shields.io/badge/MetingJS-1.2.0-green.svg)](https://github.com/metowolf/MetingJS)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

**An iOS Liquid Glass Music Player Plugin for Hexo Blogs with Draggable Edge Snapping**

[English Documentation](./README_EN.md) · [中文文档](./README.md)

</div>

---

## Table of Contents
- [✨ Key Features](#-key-features)
- [🚀 Quick Start & Installation](#-quick-start--installation)
- [⚙️ Configuration (`_config.yml`)](#️-configuration-_configyml)
- [🏷️ Tag Usage (Markdown Posts)](#️-tag-usage-markdown-posts)
- [🛠️ Developer & Fork Guide](#️-developer--fork-guide)
  - [Directory Architecture](#directory-architecture)
  - [Module Responsibilities](#module-responsibilities)
  - [Customization Cheat Sheet: Where and What to Edit](#customization-cheat-sheet-where-and-what-to-edit)
  - [Build & Local Testing Pipeline](#build--local-testing-pipeline)
- [❓ FAQ](#-faq)
- [📄 License](#-license)

---

## ✨ Key Features

1. **iOS Liquid Glass Aesthetics**:
   - Advanced multi-layer frosted glass rendering (`backdrop-filter: blur(28px) saturate(190%)`).
   - Specular border reflection highlight (`inset 0 1px 1px rgba(255,255,255,0.9)`) and fluid ambient shadows, seamlessly adapting to both Light and Dark modes.
2. **Screen-Centered Floating Lyrics Capsule HUD**:
   - Detached from the cramped bottom bar and rendered as a floating frosted pill at screen center with highlighted typography and smooth fade transitions.
   - Fully supports **free 2D mouse and touch dragging** anywhere across the screen.
3. **Free 2D Dragging & Edge Snapping Mini-Orb**:
   - **Free 2D Dragging**: Drag the player card anywhere freely with automatic viewport boundary clamping.
   - **Hemisphere Mini-Orb Docking**: Drag near the left/right screen edge (< 70px) or click the dedicated dock button `|←` to collapse into an iOS semi-circular Mini-Orb with a 360° continuously rotating vinyl album disc.
   - **Edge Vertical Sliding**: Slide the docked Mini-Orb up and down freely along the edge; pull inward toward screen center to smoothly expand.
   - **State Persistence**: Saves dock position and coordinates in `localStorage`, maintaining state across PJAX page transitions and reloads.
4. **Mobile Responsive Optimization**:
   - Bottom bar compacted to 56px height, with playlist folded by default (`list_folded: true`) to prevent covering mobile reading space.
   - Zero lyrics clipping with precise vertical centering on smartphone screens.
5. **MetingJS API Proxy Route Fix**:
   - Automatically handles query parameter template injection (`?server=:server&type=:type&id=:id&r=:r`), eliminating 404s and playback skipping issues ("An audio error has occurred").

---

## 🚀 Quick Start & Installation

### Option 1: Local Module in Hexo `source/` (Recommended)

Clone directly into your blog's `source/New-Aplayer` directory:

```bash
cd "your-hexo-blog"
git clone https://github.com/your-username/New-Aplayer.git source/New-Aplayer
cd source/New-Aplayer
npm install
npm run build
```

Then add the local file dependency in your root `package.json`:
```json
{
  "dependencies": {
    "hexo-tag-aplayer": "file:source/New-Aplayer"
  }
}
```

### Option 2: Replace NPM Package in `node_modules`

```bash
cd "your-hexo-blog"
rm -rf node_modules/hexo-tag-aplayer
git clone https://github.com/your-username/New-Aplayer.git node_modules/hexo-tag-aplayer
cd node_modules/hexo-tag-aplayer
npm install
npm run build
```

---

## ⚙️ Configuration (`_config.yml`)

Add the following configuration to your Hexo root `_config.yml`:

```yaml
# APlayer & Meting Global Settings
aplayer:
  meting: true                                  # Enable MetingJS for NetEase/QQ/Kugou music
  meting_api: https://api.injahow.cn/meting/    # Meting API URL (auto template appended)
  asset_inject: true                            # Auto inject liquid glass assets into pages
  global:
    enable: true                                # Enable global fixed bottom player
    id: 13104322073                             # Playlist / Song ID
    server: netease                             # Music server: netease / tencent / kugou
    type: playlist                              # Type: playlist / song / artist
    fixed: true                                 # Fixed to bottom of screen
    autoplay: false                             # Autoplay (recommended: false)
    order: list                                 # Order: list / random
    preload: metadata                           # Preload: metadata / auto / none
    mutex: true                                 # Single audio playback mutex
    list_folded: true                           # Fold playlist by default (recommended for mobile)
```

---

## 🏷️ Tag Usage (Markdown Posts)

### 1. Meting Playlist Tag
```markdown
{% meting "13104322073" "netease" "playlist" "listfolded:true" %}
```

### 2. Standard APlayer Song Tag
```markdown
{% aplayer "Song Title" "Artist" "https://domain.com/music.mp3" "https://domain.com/cover.jpg" "autoplay:false" %}
```

---

## 🛠️ Developer & Fork Guide

This project is built on an ES6 + Babel + Hexo Generator architecture.

### Directory Architecture

```text
New-Aplayer/
├── assets/                       # Core frontend static assets
│   ├── APlayer.glass.css         # ★ iOS Liquid Glass styling (visual customizations)
│   └── APlayer.dock.js           # ★ Dragging & edge docking engine (interaction logic)
├── common/                       # Shared constants and utility functions
│   ├── constant.es               # Markers and constant definitions
│   └── util.es                   # Option extractors and helper routines
├── lib/                          # Hexo backend rendering engine (ES6 source)
│   ├── config.es                 # Config parser and asset generator registrar
│   ├── view.es                   # View manipulator and DOM injector
│   └── tag/                      # Hexo tag parsers
│       ├── base.es               # Base tag class
│       ├── player.es             # {% aplayer %} tag parser
│       ├── playerLyric.es        # Lyric tag parser
│       ├── playerList.es         # Playlist tag parser
│       └── playerMeting.es       # ★ {% meting %} tag parser & API routing
├── scripts/                      # Build tools
│   └── build.js                  # Babel compiler (transpiles .es to .js)
├── index.es                      # ★ Plugin entrypoint & global player markup generator
├── package.json                  # Dependencies & build scripts
├── README.md                     # Chinese documentation
└── README_EN.md                  # English documentation
```

---

### Module Responsibilities

1. **`assets/APlayer.glass.css`**:
   - Manages frosted glass visual styling, specular reflection borders, Mini-Orb docked hemisphere layout, lyrics HUD capsule, and responsive media queries.
2. **`assets/APlayer.dock.js`**:
   - Handles standard Pointer Events for PC & mobile touch, 2D free dragging, edge snapping calculations, edge vertical sliding, undocking transitions, and `localStorage` caching.
3. **`lib/config.es`**:
   - Parses site `_config.yml`, registers asset generation pipeline with Hexo so that files in `assets/` automatically deploy into `public/assets/`.
4. **`index.es`**:
   - Hooks into Hexo's `after_render:html` filter, injects global fixed player markup, CSS stylesheets, and JS script tags.
5. **`lib/tag/playerMeting.es`**:
   - Parses Markdown `{% meting %}` tags and formats API endpoint query templates.

---

### Customization Cheat Sheet: Where and What to Edit

| Customization Goal | Files to Edit | Instructions |
| :--- | :--- | :--- |
| **Glass Color / Blur / Transparency** | `assets/APlayer.glass.css` | Edit `:root` variables: `--aplayer-glass-bg`, `--aplayer-glass-blur`, `--aplayer-glass-border`. |
| **Mini-Orb Dimensions / Corner Radius** | `assets/APlayer.glass.css` | Find `.aplayer-docked-left` & `.aplayer-docked-right`, adjust `width: 50px` and `border-radius`. |
| **Edge Snapping Threshold Distance** | `assets/APlayer.dock.js` | Find `snapThreshold = Math.min(70, vw * 0.18)` and adjust pixel trigger range. |
| **Custom Dock Icon** | `assets/APlayer.dock.js` | Find `injectDockButton()`, customize the SVG inner markup. |
| **Default Playlist Fold / Unfold** | `index.es` & `_config.yml` | Set `list_folded: true` in `_config.yml` or change default check in `index.es`. |
| **Custom Meting API Endpoint** | `lib/tag/playerMeting.es` | Find `formatMetingApi`, configure parameter templates. |

---

### Build & Local Testing Pipeline

After modifying any `.es` source file or asset, compile and test with the following commands:

```bash
# 1. Run Babel transpiler inside New-Aplayer
cd source/New-Aplayer
npm run build

# 2. Clean and regenerate Hexo static files
cd ../..
npx hexo clean && npx hexo generate

# 3. Start local development server
npx hexo server -p 4000
```

---

## ❓ FAQ

#### Q1: Why does the console show "An audio error has occurred, player will skip forward..."?
**A**: This occurs when third-party Meting APIs lack query templates or require VIP credentials. This plugin automatically appends `:server/:type/:id`, ensuring proper queries. Verify your `_config.yml` points to an active `meting_api` (e.g. `https://api.injahow.cn/meting/`).

#### Q2: Does the player remember its position on mobile after page transitions?
**A**: Yes. Position coordinates and docking side are stored in `localStorage`, maintaining state across PJAX page loads and browser refreshes.

#### Q3: Why didn't my edits in `lib/*.es` take effect?
**A**: You must run `npm run build` in `source/New-Aplayer` to transpile ES6 `.es` files into CommonJS `.js` files before generating Hexo.

---

## 📄 License

Open-sourced under the [MIT License](LICENSE).
