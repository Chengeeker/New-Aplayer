# New-Aplayer 临时交接总结

## 1. 用户需求

用户博客使用 `MoePlayer/hexo-tag-aplayer`，但原插件长期没有维护，Butterfly 新版本也移除了 APlayer/MetingJS 支持。用户希望维护自己的版本，目录固定为：

```text
D:\Blog Center\Chen blog\source\New-Aplayer
```

需求包括：

1. 支持最新 Hexo 的全局吸底播放器。
2. 使用 iOS 液态玻璃风格。
3. 深色和浅色模式下都保持可读性。
4. 尽量只使用 CSS 和 JavaScript，不引入 UI 组件库。
5. 保留原有 APlayer、MetingJS 和 Hexo 标签功能。
6. 项目目录始终保持为 `New-Aplayer`，不改名。
7. 方便独立上传 GitHub，也方便其他人下载使用。
8. Hexo 启动时不能因为项目依赖文件产生 YAML/JSON 解析错误。

用户提供的网易云歌单：

```text
https://music.163.com/playlist?id=13104322073
```

参数为：

```text
id: 13104322073
server: netease
type: playlist
```

## 2. 已完成的工作

### 2.1 项目基线

已将上游 `hexo-tag-aplayer` 源码放入 `New-Aplayer`，保留原有标签：

- `{% aplayer %}`
- `{% aplayerlrc %}`
- `{% aplayerlist %}`
- `{% meting %}`

项目版本调整为 `3.1.0`，入口仍然是 `index.js`。

### 2.2 全局吸底播放器

新增插件级全局配置。插件会在完整页面的 `</body>` 前生成播放器，并自动加载 APlayer、MetingJS、原始 APlayer CSS 和玻璃 CSS：

```yml
aplayer:
  meting: true
  asset_inject: true
  global:
    enable: true
    id: 13104322073
    server: netease
    type: playlist
    fixed: true
    autoplay: false
    order: list
    preload: metadata
    mutex: true
```

插件会自动生成带有 `aplayer-global-marker` 和 `aplayer-fixed` 的播放器容器。

同时兼容以下原生写法：

```html
<div class="aplayer"
     data-id="..."
     data-server="netease"
     data-type="playlist"
     data-fixed="true">
</div>
```

```html
<meting-js id="..." server="netease" type="playlist" fixed="true"></meting-js>
```

### 2.3 液态玻璃样式

新增文件：

```text
assets/APlayer.glass.css
```

包含：

- 半透明玻璃背景
- `backdrop-filter: blur(...)`
- 深色/浅色模式变量
- `html[data-theme="dark"]` 识别
- `prefers-color-scheme: dark` 识别
- Butterfly 页面环境兼容
- 移动端宽度和安全区域适配
- 不支持毛玻璃浏览器时的背景回退
- 播放列表、按钮、歌词、作者和进度条的可读性处理

### 2.4 修复“只有一条细线”的问题

浏览器实际检查发现，播放器 DOM 已生成，但 APlayer/MetingJS 的 JS 没有执行。原因是 Windows `path.join()` 生成了反斜杠 URL：

```text
/%5Cassets%5Cjs%5CAPlayer.min.js
/%5Cassets%5Cjs%5CMeting.min.js
```

现在增加了 URL 规范化：

```js
const normalizeUrlPath = value => value.replace(/\\/g, '/')
```

正确地址应为：

```text
/assets/js/APlayer.min.js
/assets/js/Meting.min.js
/assets/css/APlayer.min.css
/assets/css/APlayer.glass.css
```

APlayer fixed 模式默认还会添加 `aplayer-narrow`，原始 CSS 会把播放器收缩成窄标签。玻璃 CSS 现在会覆盖该行为：

- 桌面端高度约 66px
- 桌面端宽度约 420px
- 移动端使用 `calc(100vw - 16px)`
- 主体、标题、进度条和控制按钮保持可见
- 播放列表仍可通过菜单按钮展开

### 2.5 Hexo 依赖目录报错修复

本地包通过 `file:source/New-Aplayer` 安装时，npm 可能生成：

```text
source/New-Aplayer/node_modules
```

Hexo 会扫描 `source` 下的文件，把依赖包 README 当作文章解析，产生：

```text
YAMLException: a line break is expected
```

博客根目录 `_config.yml` 已加入：

```yml
ignore:
  - New-Aplayer/node_modules/**
  - source/New-Aplayer/node_modules/**
```

即使 npm 重新生成嵌套依赖，Hexo 也不会再扫描它们。

### 2.6 Hexo 8 和现代依赖兼容

修复了上游旧代码与现代依赖的兼容问题：

- `hexo-log` 新版可能导出对象或 `default`，不再假设它一定是函数。
- `hexo-fs` 改为命名空间导入：`import * as fs from 'hexo-fs'`。
- `hexo-util` 改为命名空间导入：`import * as util from 'hexo-util'`。
- Windows 资源部署路径和浏览器 URL 分开处理，URL 统一使用正斜杠。

## 3. 具体修改的文件

### 3.1 New-Aplayer 项目内

```text
D:\Blog Center\Chen blog\source\New-Aplayer\.gitignore
D:\Blog Center\Chen blog\source\New-Aplayer\README.md
D:\Blog Center\Chen blog\source\New-Aplayer\package.json
D:\Blog Center\Chen blog\source\New-Aplayer\index.es
D:\Blog Center\Chen blog\source\New-Aplayer\index.js
D:\Blog Center\Chen blog\source\New-Aplayer\lib\config.es
D:\Blog Center\Chen blog\source\New-Aplayer\lib\config.js
D:\Blog Center\Chen blog\source\New-Aplayer\lib\tag\playerMeting.es
D:\Blog Center\Chen blog\source\New-Aplayer\lib\tag\playerMeting.js
D:\Blog Center\Chen blog\source\New-Aplayer\common\constant.es
D:\Blog Center\Chen blog\source\New-Aplayer\common\constant.js
D:\Blog Center\Chen blog\source\New-Aplayer\common\util.js
D:\Blog Center\Chen blog\source\New-Aplayer\assets\APlayer.glass.css
D:\Blog Center\Chen blog\source\New-Aplayer\scripts\build.js
```

### 3.2 博客根目录

```text
D:\Blog Center\Chen blog\_config.yml
D:\Blog Center\Chen blog\package.json
D:\Blog Center\Chen blog\package-lock.json
```

博客根目录 `package.json` 使用本地项目：

```json
"hexo-tag-aplayer": "file:source/New-Aplayer"
```

博客根目录 `_config.yml` 已设置：

```yml
asset_inject: true
```

并写入了网易云歌单全局播放器配置和 `ignore` 规则。

## 4. 如何复刻这些操作

### 4.1 安装/同步依赖

在博客根目录执行：

```powershell
cd "D:\Blog Center\Chen blog"
npm install --legacy-peer-deps
```

不要在 `source/New-Aplayer` 内单独执行 `npm install`。如果 npm 生成了嵌套 `node_modules`，根目录 `_config.yml` 的 `ignore` 规则会阻止 Hexo 扫描它。

### 4.2 构建插件

```powershell
cd "D:\Blog Center\Chen blog\source\New-Aplayer"
npm run build
```

### 4.3 清理和启动 Hexo

```powershell
cd "D:\Blog Center\Chen blog"
hexo clean
hexo s
```

如果 4000 端口是旧进程，可以使用：

```bash
hexo s --port 4020
```

浏览器使用 `Ctrl + F5`，或执行“清空缓存并硬性重新加载”。

### 4.4 检查资源地址

开发者工具 Network 中应看到：

```text
/assets/js/APlayer.min.js
/assets/js/Meting.min.js
/assets/css/APlayer.min.css
/assets/css/APlayer.glass.css
```

不能出现：

```text
/%5Cassets%5Cjs%5CAPlayer.min.js
```

### 4.5 检查播放器尺寸

DOM 中应看到：

```html
<div class="aplayer ... aplayer-global-marker ... aplayer-fixed ...">
```

正常情况下播放器主体高度约为 66px。如果只有 1～2px，优先检查：

1. APlayer JS 是否成功加载。
2. MetingJS 是否成功加载。
3. 脚本 URL 是否出现 `%5C`。
4. 是否仍在使用旧 Hexo 进程。
5. 浏览器是否使用了旧缓存。

## 5. 当前验证结果

最后一次浏览器验证使用临时端口 `4020`：

```text
APlayer.min.js       已加载
Meting.min.js        已加载
全局播放器容器      已生成
网易云歌单 ID        已存在
播放器外层高度      约 66px
播放器主体高度      约 67px
fixed 模式          已生效
```

当前已经解决：

- 原插件未被博客正确替换的问题
- Hexo 扫描 `New-Aplayer/node_modules` 导致的 YAML 报错
- Hexo 8 依赖导出兼容问题
- Windows 反斜杠资源 URL 问题
- APlayer fixed/narrow 模式压缩成细线问题

后续如果继续优化，建议先进行真实浏览器截图检查，再微调玻璃透明度、阴影、圆角、列表展开方式和移动端布局。
