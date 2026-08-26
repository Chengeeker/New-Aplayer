# New-Aplayer

<div align="center">

[![Hexo](https://img.shields.io/badge/Hexo-5.0+-blue.svg)](https://hexo.io/)
[![APlayer](https://img.shields.io/badge/APlayer-1.10.1-red.svg)](https://aplayer.js.org/)
[![MetingJS](https://img.shields.io/badge/MetingJS-1.2.0-green.svg)](https://github.com/metowolf/MetingJS)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

**为 Hexo 博客定制的毛玻璃拟态、贴边吸附与悬浮歌词 APlayer / Meting 音乐播放器插件**

[English Documentation](./README_EN.md) · [中文文档](./README.md)

</div>

---

## 目录
- [✨ 功能特性](#-功能特性)
- [💻 环境支持与验证](#-环境支持与验证)
- [🚀 快速开始与安装](#-快速开始与安装)
- [⚙️ 配置说明 (`_config.yml`)](#️-配置说明-_configyml)
- [⚠️ 关键避坑与自检提示](#️-关键避坑与自检提示)
- [🏷️ 标签用法 (Tag Plugins)](#️-标签用法-tag-plugins)
- [🛠️ 二次开发与 Fork 定制指南](#️-二次开发与-fork-定制指南)
  - [项目目录结构](#项目目录结构)
  - [核心代码模块职责](#核心代码模块职责)
  - [修改指引：想改什么、改哪个文件](#修改指引想改什么改哪个文件)
  - [构建与本地测试流程](#构建与本地测试流程)
- [❓ 常见问题与排查 (FAQ)](#-常见问题与排查-faq)
- [📄 开源协议](#-开源协议)

---

## ✨ 功能特性

1. **毛玻璃拟态风格（Frosted Glass）**：
   - 采用多层高斯模糊与半透明自适应背景，搭配细腻微光边框与柔和阴影，与博客浅色/深色（Dark Mode）自然融合。
2. **全局悬浮歌词胶囊 HUD**：
   - 歌词脱离播放器狭窄底栏，独立置于屏幕中央上方呈胶囊状展示，支持单句高光与双语歌词。
   - 歌词胶囊支持全屏幕鼠标/触屏 **2D 自由拖拽**。
3. **全自由 2D 拖拽 + 精准贴边吸附**：
   - **自由 2D 拖拽**：按住播放器可在屏幕任意位置随意拖拽停放，视口边界自动保护。
   - **严格贴边吸附（8px）**：仅在播放器被完全推到紧贴屏幕边缘（<= 8px）或点击专属贴边按钮 `|←` 时，才折叠为半球 Mini-Orb，彻底杜绝远距离误吸附。
   - **边缘沿轴滑动**：吸附状态下可沿屏幕边缘上下滑动调节位置；向屏幕内侧拖拽即可顺滑展开。
   - **状态持久化记忆**：基于 `localStorage` 自动记录坐标与吸附状态，刷新页面与 PJAX 切页后自动精准恢复。
4. **毫秒级本地缓存加载（Meting Fast-Cache v2）**：
   - 自动在本地缓存歌单元数据（API 节点 Hash 绑定与存储配额保护），刷新页面或切页时实现 0 秒毫秒级瞬间渲染，杜绝长时间空白等待。
5. **移动端深度响应式适配**：
   - 播放器底栏高度优化为紧凑 56px，默认自动折叠歌单（`list_folded: true`），避免遮挡移动端阅读区域。
   - 消除歌词基线裁切，在小屏手机上垂直精准居中。
6. **Meting API 路由修复与标签语法增强**：
   - 自动补全 API 参数占位符（`:server/:type/:id`），避免 404 与歌曲跳过报错（“An audio error has occurred”）。
   - 标签解析全面兼容 `autoplay:false`、`listfolded:true`、`fixed:true` 等键值与布尔写法。

---

## 💻 环境支持与验证

本插件已在以下主流生产环境中完成完整集成与兼容性验证：

- **Hexo 版本**：Hexo `5.x` / `6.x` / `7.x` / `8.x`
- **Node.js 版本**：Node.js `16.x` / `18.x` / `20.x` / `22.x`
- **支持主题**：Butterfly（`4.x` / `5.x`）、Fluid、Anzhiyu、NexT 等主流现代主题
- **特性支持**：PJAX 无刷新跳转、深浅色模式自适应切换、移动端触摸手势

---

## 🚀 快速开始与安装

本项目采用标准的 Hexo 本地源码模块集成方案，预编译好的 CommonJS 运行文件已直接包含在仓库中，开箱即用：

> 💡 **完全独立闭环（无需安装原版 APlayer 插件）**：
> 本项目为完整的一站式独立插件，所有渲染标签、生成管道、样式及脚本均已全量内置，**新站点完全不需要提前安装原版 `hexo-tag-aplayer` 或任何前置音乐插件**。

### 第一步：克隆源码至博客的 `source` 目录

在你的 **Hexo 博客根目录** 下执行：

```bash
git clone https://github.com/Chengeeker/New-Aplayer.git source/New-Aplayer
```

### 第二步：在博客根目录关联本地依赖

在你的 **Hexo 博客根目录** 的 `package.json` 中配置本地依赖：

```json
{
  "dependencies": {
    "hexo-tag-aplayer": "file:source/New-Aplayer"
  }
}
```

然后在 **Hexo 博客根目录** 下执行一次安装：

```bash
npm install --legacy-peer-deps
```

> **重要提示**：
> 1. 请务必在 **Hexo 博客根目录** 下执行安装，不要在 `source/New-Aplayer/` 内部直接生成 `node_modules`，以防 Hexo 错误扫描其内部文件；
> 2. 推荐使用 `--legacy-peer-deps`，可有效避免 npm 严格校验博客中其他年代久远的第三方旧插件时抛出 `ERESOLVE` 依赖冲突。

---

## ⚙️ 配置说明 (`_config.yml`)

在 Hexo 站点根目录的 **`_config.yml`**（注意是站点配置文件，不是主题配置文件）中添加如下配置：

```yaml
# APlayer & Meting 全局配置
aplayer:
  meting: true                                  # 必须为 true：启用 MetingJS 解析网易云/QQ音乐歌单
  asset_inject: true                            # 必须为 true：自动将样式与脚本注入页面（若设为 false 则完全不显示）
  meting_api: https://api.injahow.cn/meting/    # Meting API 节点（此处为公共测试接口，生产建议自建）
  global:
    enable: true                                # 开启全站固定底栏播放器
    id: 13104322073                             # 歌单 ID / 歌曲 ID（请替换为你自己的歌单 ID）
    server: netease                             # 音乐平台: netease(网易云) / tencent(QQ音乐) / kugou(酷狗)
    type: playlist                              # 类型: playlist(歌单) / song(单曲) / artist(歌手)
    fixed: true                                 # 吸底模式 (固定在页面底部)
    autoplay: false                             # 是否自动播放 (由于浏览器策略限制，默认建议 false)
    order: list                                 # 播放顺序: list(列表) / random(随机)
    preload: metadata                           # 预加载策略: metadata / auto / none
    mutex: true                                 # 互斥播放: 同时只允许一个播放器发声
    list_folded: true                           # 默认折叠歌单列表（推荐 true，防止移动端遮挡）
```

> ⚠️ **关于 Meting API 的说明**：
> `https://api.injahow.cn/meting/` 仅为公共演示接口，公共接口可能会受到网络波动或服务变动影响。对于长期稳定使用的博客，**建议自行部署 Meting-API 服务**（基于 Vercel、Docker 或自建服务器），并将地址填入 `meting_api`。

---

## ⚠️ 关键避坑与自检提示

如果安装后播放器没有显示，请对照以下几点快速自查：

1. **`asset_inject` 必须设为 `true`**：
   - 若写成 `asset_inject: false`，插件将完全跳过所有 CSS/JS 资源与播放器 DOM 的注入。
2. **必须保留 `global:` 配置块**：
   - 只有声明了 `global:` 块及其内部的 `id`、`server`、`type`，插件才会为全站生成底栏播放器。
3. **关闭主题自带的 APlayer（防止冲突）**：
   - 如果你的主题（如 Butterfly 的 `_config.butterfly.yml` 或 Fluid 的配置）中带有内置的 `aplayer: enable: true`，请**将其关闭（设为 `false`）**，因为本插件已经全自动接管全局注入，关闭主题内置项可避免双重加载与脚本冲突。
4. **修改配置后务必清理缓存重建**：
   - 每次修改 `_config.yml` 后，必须在博客根目录执行 `npx hexo clean && npx hexo generate` 重新生成静态页面。

---

## 🏷️ 标签用法 (Tag Plugins)

除了全局吸底播放器，你还可以在任意 Markdown 文章中使用标签插入独立播放器：

### 1. Meting 平台歌单标签
```markdown
{% meting "13104322073" "netease" "playlist" "listfolded:true" "autoplay:false" %}
```

### 2. 标准 APlayer 单曲标签
```markdown
{% aplayer "歌曲名称" "歌手名" "https://domain.com/music.mp3" "https://domain.com/cover.jpg" "autoplay:false" %}
```

---

## 🛠️ 二次开发与 Fork 定制指南

本项目采用标准的 ES6 + Babel + Hexo Generator 架构，代码结构清晰，任何人 Fork 之后都可以轻松上手进行二次定制。

### 项目目录结构

```text
New-Aplayer/
├── assets/                       # 核心静态资源库（前端直引资源）
│   ├── APlayer.glass.css         # ★ 毛玻璃样式表（外观定制）
│   └── APlayer.dock.js           # ★ 拖拽、贴边吸附与缓存引擎（交互定制）
├── common/                       # 公共常量与工具函数
│   ├── constant.es               # 常量定义与资源标记 Marker
│   └── util.es                   # 选项提取、安全转义与工具函数
├── lib/                          # 后端渲染引擎（Babel ES6 源码）
│   ├── config.es                 # Hexo 配置解析与静态资源注册器
│   ├── view.es                   # HTML 视图资源注入与 DOM 操作
│   └── tag/                      # Hexo 标签解析器
│       ├── base.es               # 标签基础类
│       ├── player.es             # {% aplayer %} 标签渲染
│       ├── playerLyric.es        # 歌词标签渲染
│       ├── playerList.es         # 播放列表标签渲染
│       └── playerMeting.es       # ★ {% meting %} 标签解析与 API 路由
├── scripts/                      # 构建与测试工具
│   ├── build.js                  # Babel 编译脚本 (ES6 .es -> CommonJS .js)
│   └── test.js                   # 自动化单元与集成测试套件
├── index.es                      # ★ 插件总入口文件 (Hexo 过滤器与全局播放器生成)
├── package.json                  # 项目依赖与编译脚本
├── README.md                     # 中文说明文档
└── README_EN.md                  # 英文说明文档
```

---

### 核心代码模块职责

1. **`assets/APlayer.glass.css`**：
   - 负责播放器底栏、吸附半球 Mini-Orb、悬浮歌词胶囊 HUD、播放列表卡片的毛玻璃半透明风格、微光边框及媒体查询。
2. **`assets/APlayer.dock.js`**：
   - 负责鼠标与触摸手势捕获、2D 自由拖拽、8px 边缘吸附判定、边缘滑动、Meting 本地缓存加速与 `localStorage` 状态持久化。
3. **`lib/config.es`**：
   - 负责读取站点 `_config.yml` 中的 `aplayer` 配置项，向 Hexo 注册静态资产生成管道（使得 `assets/` 下的 CSS/JS 自动生成到博客的 `public/assets/` 目录中）。
4. **`index.es`**：
   - 插件主入口，负责在页面 HTML 渲染阶段（`after_render:html`）注入全局固定播放器 DOM、样式链接及脚本。
5. **`lib/tag/playerMeting.es`**：
   - 负责解析文章中的 `{% meting %}` 标签，格式化 API 模板与参数清洗，避免跨域或 404。

---

### 修改指引：想改什么、改哪个文件

| 定制诉求 | 涉及修改的文件 | 具体修改位置与说明 |
| :--- | :--- | :--- |
| **修改毛玻璃颜色/模糊度/透明度** | `assets/APlayer.glass.css` | 修改 `:root` 变量 `--aplayer-glass-bg`、`--aplayer-glass-blur`、`--aplayer-glass-border` 等。 |
| **修改贴边半球的尺寸/圆角** | `assets/APlayer.glass.css` | 搜索 `.aplayer-docked-left` 与 `.aplayer-docked-right`，修改 `width: 50px`、`border-radius: 0 28px 28px 0`。 |
| **调整吸附灵敏度阈值** | `assets/APlayer.dock.js` | 搜索 `snapThreshold = 8`，调整触发贴边的像素距离。 |
| **修改贴边按钮图标** | `assets/APlayer.dock.js` | 搜索 `injectDockButton()` 函数，替换内部的 `btn.innerHTML` SVG 矢量图形。 |
| **修改默认是否折叠歌单** | `index.es` & `_config.yml` | 在 `index.es` 中修改 `data-listfolded` 默认值，或在 `_config.yml` 中设置 `list_folded: true`。 |
| **调整 Meting 缓存过期时间** | `assets/APlayer.dock.js` | 搜索 `CACHE_EXPIRY_MS`（默认为 24 小时）。 |

---

### 构建与本地测试流程

如果你对 `.es` 源代码或 `assets/` 资源进行了二次开发，按以下步骤进行编译与测试：

```bash
# 1. 在 New-Aplayer 目录下运行编译与全套集成测试
cd source/New-Aplayer
npm run build
npm test

# 2. 返回 Hexo 博客根目录，清理并重新生成静态文件
cd ../..
npx hexo clean && npx hexo generate

# 3. 启动本地预览服务器
npx hexo server -p 4000
```

---

## ❓ 常见问题与排查 (FAQ)

#### Q1: 为什么控制台提示 "An audio error has occurred, player will skip forward..."？
**A**: 这是由于部分三方 Meting API 未正确配置路由模板或歌曲需要平台会员权限。本项目已内置参数模板补全（`:server/:type/:id`），请确保 `_config.yml` 中配置了可用的 `meting_api` 地址。

#### Q2: 手机端切换页面或刷新后播放器位置会保持吗？
**A**: 会。播放器与贴边半球的坐标及吸附状态均已自动持久化存入浏览器的 `localStorage`，初次进入页面或通过 PJAX 无刷新切页均会自动恢复保存的位置。

#### Q3: 为什么修改了 `lib/*.es` 代码后 Hexo 博客没有生效？
**A**: 必须在 `source/New-Aplayer` 目录下运行 `npm run build`，将 ES6 源码编译为 Node.js 可执行的 CommonJS 文件（`.js`），然后重新运行 `hexo clean && hexo generate`。

#### Q4: 我的 Hexo 博客以前从未安装过 APlayer 插件，需要先装老版本吗？
**A**: **完全不需要！** `New-Aplayer` 是一个 100% 独立且闭环的完整插件包，所有标签解析、依赖库与毛玻璃资源均已全量内置。新站点只需按照上方三步安装即可直接运行。

---

## 📄 开源协议

本项目基于 [MIT License](LICENSE) 开源，欢迎提交 Issue 与 Pull Request！
