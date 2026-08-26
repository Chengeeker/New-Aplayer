# New-Aplayer (Frosted Glass & Docking Edition)

<div align="center">

[![Hexo](https://img.shields.io/badge/Hexo-5.0+-blue.svg)](https://hexo.io/)
[![APlayer](https://img.shields.io/badge/APlayer-1.10.1-red.svg)](https://aplayer.js.org/)
[![MetingJS](https://img.shields.io/badge/MetingJS-1.2.0-green.svg)](https://github.com/metowolf/MetingJS)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

**A Frosted-Glass APlayer / Meting Plugin for Hexo with Draggable Edge Snapping & Lyrics HUD**

[English Documentation](./README_EN.md) · [中文文档](./README.md)

</div>

---

## Table of Contents
- [✨ Key Features](#-key-features)
- [💻 Environment Support & Verification](#-environment-support--verification)
- [🚀 Quick Start & Installation](#-quick-start--installation)
- [⚙️ Configuration (`_config.yml`)](#️-configuration-_configyml)
- [⚠️ Key Pitfalls & Troubleshooting Checklist](#️-key-pitfalls--troubleshooting-checklist)
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

1. **Frosted Glass Aesthetics**:
   - Multi-layer Gaussian blur with semi-transparent adaptive backgrounds, subtle highlight borders, and soft fluid shadows that blend smoothly with both Light and Dark themes.
2. **Screen-Centered Floating Lyrics Capsule HUD**:
   - Detached from the cramped bottom bar and rendered as a floating frosted pill at screen center with highlighted typography.
   - Fully supports **free 2D mouse and touch dragging** anywhere across the screen.
3. **Free 2D Dragging & Strict Edge Snapping (8px)**:
   - **Free 2D Dragging**: Drag the player card anywhere freely with automatic viewport boundary protection.
   - **Strict Edge Snapping (8px)**: Only collapses into the semi-circular Mini-Orb when physically pushed flush against the screen edge (<= 8px) or clicking the dedicated dock button `|←`, preventing accidental premature snapping from a distance.
   - **Edge Vertical Sliding**: Slide the docked Mini-Orb up and down freely along the edge; pull inward toward screen center to smoothly expand.
   - **State Persistence & Restoration**: Saves dock coordinates and state in `localStorage`, automatically restoring position upon initial load, refresh, and PJAX transitions.
4. **Millisecond-Level Local Caching (Meting Fast-Cache v2)**:
   - Automatically caches playlist metadata in `localStorage` with API key hashing and quota overflow protection, eliminating network latency and rendering the player instantly in 0 milliseconds upon page refreshes and navigation.
5. **Mobile Responsive Optimization**:
   - Bottom bar compacted to 56px height, with playlist folded by default (`list_folded: true`) to prevent covering mobile reading space.
   - Zero lyrics clipping with precise vertical centering on smartphone screens.
6. **Meting API Proxy Route Fix & Tag Syntax Enhancements**:
   - Automatically handles query parameter template injection (`?server=:server&type=:type&id=:id&r=:r`), eliminating 404s and playback skipping issues ("An audio error has occurred").
   - Full tag parser compatibility with `autoplay:false`, `listfolded:true`, `fixed:true`, etc.

---

## 💻 Environment Support & Verification

This plugin has been integrated and verified across modern production environments:

- **Hexo Version**: Hexo `5.x` / `6.x` / `7.x` / `8.x`
- **Node.js Version**: Node.js `16.x` / `18.x` / `20.x` / `22.x`
- **Theme Compatibility**: Butterfly (`4.x` / `5.x`), Fluid, Anzhiyu, NexT, and other modern Hexo themes
- **Features Tested**: PJAX transitions, Light/Dark mode auto-switching, and Mobile touch dragging

---

## 🚀 Quick Start & Installation

This project adopts a standardized Hexo local source module integration approach. Pre-compiled CommonJS runtime files are included directly in the repository:

### Step 1: Clone source into your blog's `source` folder

Run the following in your **Hexo blog root**:

```bash
git clone https://github.com/Chengeeker/New-Aplayer.git source/New-Aplayer
```

### Step 2: Link Local Dependency in Root `package.json`

Edit the root `package.json` of your **Hexo blog**, adding the local file dependency:

```json
{
  "dependencies": {
    "hexo-tag-aplayer": "file:source/New-Aplayer"
  }
}
```

Then install the dependency link from your **Hexo blog root**:

```bash
npm install --legacy-peer-deps
```

> **Important Notes**:
> 1. Always run `npm install` from the **Hexo blog root**, rather than inside `source/New-Aplayer/`, to prevent Hexo from mistakenly scanning nested `node_modules` inside the source directory.
> 2. Using `--legacy-peer-deps` avoids `ERESOLVE` errors when your Hexo blog contains older legacy plugins.

---

## ⚙️ Configuration (`_config.yml`)

Add the following configuration to your Hexo site root **`_config.yml`** (note: site config, not theme config):

```yaml
# APlayer & Meting Global Settings
aplayer:
  meting: true                                  # Required: enable MetingJS for NetEase/QQ/Kugou music
  asset_inject: true                            # Required: auto inject assets into pages (if false, player will NOT display)
  meting_api: https://api.injahow.cn/meting/    # Meting API URL (public demo endpoint)
  global:
    enable: true                                # Enable global fixed bottom player
    id: 13104322073                             # Playlist / Song ID (replace with your own playlist ID)
    server: netease                             # Music server: netease / tencent / kugou
    type: playlist                              # Type: playlist / song / artist
    fixed: true                                 # Fixed to bottom of screen
    autoplay: false                             # Autoplay (recommended: false)
    order: list                                 # Order: list / random
    preload: metadata                           # Preload: metadata / auto / none
    mutex: true                                 # Single audio playback mutex
    list_folded: true                           # Fold playlist by default (recommended for mobile)
```

> ⚠️ **Note on Meting API**:
> `https://api.injahow.cn/meting/` is provided as a public demo endpoint. Public endpoints may be subject to network latency or rate limits. For production and long-term stability, **it is recommended to self-host a Meting-API instance** (via Docker, Vercel, or personal server) and configure its URL in `meting_api`.

---

## ⚠️ Key Pitfalls & Troubleshooting Checklist

If the player does not appear after setup, verify the following checklist:

1. **`asset_inject` must be set to `true`**:
   - If `asset_inject: false` is configured, Hexo skips all CSS/JS stylesheet and player DOM injection completely.
2. **`global:` configuration block is required for full-site bottom player**:
   - Without the `global:` section (containing `id`, `server`, `type`), Hexo cannot determine which playlist to mount.
3. **Disable Theme Built-in APlayer (avoid conflicts)**:
   - If your theme (such as Butterfly's `_config.butterfly.yml` or Fluid's config) has an `aplayer: enable: true` setting, **turn it off (set to `false`)**. This plugin handles global asset injection automatically; disabling the theme option prevents duplicate scripts and styling clashes.
4. **Always clean and regenerate after editing configs**:
   - Run `npx hexo clean && npx hexo generate` in your blog root whenever you update `_config.yml`.

---

## 🏷️ Tag Usage (Markdown Posts)

### 1. Meting Playlist Tag
```markdown
{% meting "13104322073" "netease" "playlist" "listfolded:true" "autoplay:false" %}
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
│   ├── APlayer.glass.css         # ★ Frosted glass styling (visual customizations)
│   └── APlayer.dock.js           # ★ Dragging, edge docking & caching engine (interaction logic)
├── common/                       # Shared constants and utility functions
│   ├── constant.es               # Markers and constant definitions
│   └── util.es                   # Option extractors, safe HTML escaping & helpers
├── lib/                          # Hexo backend rendering engine (ES6 source)
│   ├── config.es                 # Config parser and asset generator registrar
│   ├── view.es                   # View manipulator and DOM injector
│   └── tag/                      # Hexo tag parsers
│       ├── base.es               # Base tag class
│       ├── player.es             # {% aplayer %} tag parser
│       ├── playerLyric.es        # Lyric tag parser
│       ├── playerList.es         # Playlist tag parser
│       └── playerMeting.es       # ★ {% meting %} tag parser & API routing
├── scripts/                      # Build and testing tools
│   ├── build.js                  # Babel compiler (transpiles .es to .js)
│   └── test.js                   # Automated unit and integration test suite
├── index.es                      # ★ Plugin entrypoint & global player markup generator
├── package.json                  # Dependencies & build scripts
├── README.md                     # Chinese documentation
└── README_EN.md                  # English documentation
```

---

### Module Responsibilities

1. **`assets/APlayer.glass.css`**:
   - Manages frosted glass visual styling, border highlights, Mini-Orb docked hemisphere layout, lyrics HUD capsule, and responsive media queries.
2. **`assets/APlayer.dock.js`**:
   - Handles standard Pointer Events for PC & mobile touch, 2D free dragging, 8px strict edge snapping calculations, edge vertical sliding, Meting caching acceleration, and `localStorage` persistence.
3. **`lib/config.es`**:
   - Parses site `_config.yml`, registers asset generation pipeline with Hexo so that files in `assets/` automatically deploy into `public/assets/`.
4. **`index.es`**:
   - Hooks into Hexo's `after_render:html` and `after_post_render` filters, injects global fixed player markup, CSS stylesheets, and JS script tags safely.
5. **`lib/tag/playerMeting.es`**:
   - Parses Markdown `{% meting %}` tags, cleans parameters, and formats API endpoint query templates.

---

### Customization Cheat Sheet: Where and What to Edit

| Customization Goal | Files to Edit | Instructions |
| :--- | :--- | :--- |
| **Glass Color / Blur / Transparency** | `assets/APlayer.glass.css` | Edit `:root` variables: `--aplayer-glass-bg`, `--aplayer-glass-blur`, `--aplayer-glass-border`. |
| **Mini-Orb Dimensions / Corner Radius** | `assets/APlayer.glass.css` | Find `.aplayer-docked-left` & `.aplayer-docked-right`, adjust `width: 50px` and `border-radius`. |
| **Edge Snapping Threshold Distance** | `assets/APlayer.dock.js` | Find `snapThreshold = 8` and adjust pixel trigger range. |
| **Custom Dock Icon** | `assets/APlayer.dock.js` | Find `injectDockButton()`, customize the SVG inner markup. |
| **Default Playlist Fold / Unfold** | `index.es` & `_config.yml` | Set `list_folded: true` in `_config.yml` or change default check in `index.es`. |
| **Meting Cache Expiration Duration** | `assets/APlayer.dock.js` | Search `CACHE_EXPIRY_MS` (default is 24 hours). |

---

### Build & Local Testing Pipeline

After modifying any `.es` source file or asset, compile and test with the following commands:

```bash
# 1. Run Babel transpiler and full test suite inside New-Aplayer
cd source/New-Aplayer
npm run build
npm test

# 2. Clean and regenerate Hexo static files
cd ../..
npx hexo clean && npx hexo generate

# 3. Start local development server
npx hexo server -p 4000
```

---

## ❓ FAQ

#### Q1: Why does the console show "An audio error has occurred, player will skip forward..."?
**A**: This occurs when third-party Meting APIs lack query templates or require VIP credentials. This plugin automatically appends `:server/:type/:id`, ensuring proper queries. Verify your `_config.yml` points to an active `meting_api`.

#### Q2: Does the player remember its position on mobile after page transitions?
**A**: Yes. Position coordinates and docking state are stored in `localStorage` and automatically restored upon initialization and PJAX page loads.

#### Q3: Why didn't my edits in `lib/*.es` take effect?
**A**: You must run `npm run build` in `source/New-Aplayer` to transpile ES6 `.es` files into CommonJS `.js` files before generating Hexo.

---

## 📄 License

Open-sourced under the [MIT License](LICENSE).
