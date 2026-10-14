
# DeepSeek Chat Tagger

[![GitHub Repo](https://img.shields.io/badge/GitHub-HU--SHD-blue.svg)](https://github.com/HU-SHD/deepseek-chat-tagger)
[![License](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![JavaScript](https://img.shields.io/badge/JavaScript-ES2020+-yellow.svg)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Manifest V3](https://img.shields.io/badge/Manifest-V3-orange.svg)](https://developer.chrome.com/docs/extensions/mv3/intro/)
[![Firefox](https://img.shields.io/badge/Firefox-109%2B-red.svg)](https://www.mozilla.org/firefox/)

> **DeepSeek Chat Tagger** 是一款为 DeepSeek 网页版（chat.deepseek.com）设计的跨浏览器扩展，用于在对话消息右上角添加三态颜色标签（默认/绿色/红色），并同步更新右侧缩略导航条颜色，帮助用户快速标记、分类和定位重要对话内容。

---

## 目录

- [项目简介](#项目简介)
- [核心特性](#核心特性)
- [快速开始](#快速开始)
- [快捷键与操作说明](#快捷键与操作说明)
- [使用示例](#使用示例)
- [源码构建](#源码构建)
- [项目结构](#项目结构)
- [核心实现原理](#核心实现原理)
- [许可证](#许可证)
- [变更日志](#变更日志)

---

## 项目简介

`DeepSeek Chat Tagger` 是一个基于 **Manifest V3** 的浏览器扩展，兼容 Chrome、Edge、Firefox、Opera 以及联想浏览器等主流 Chromium 内核浏览器。扩展通过 Content Script 注入的方式，在 DeepSeek 网页版右侧每条对话消息的右上角生成一个可交互的彩色圆点，用户按住 `Ctrl`（Mac 为 `Cmd`）并左键点击即可循环切换颜色（灰 → 绿 → 红 → 灰）。标记状态通过浏览器本地存储持久化，刷新页面后自动恢复。同时，右侧缩略导航条上的小点会同步显示对应消息的标记颜色，实现快速定位。

---

## 核心特性

- **消息级标记**：右侧每一轮对话（用户消息 / AI 回复）右上角显示一个可交互的彩色圆点。
- **三态循环**：按住 `Ctrl`（Mac 为 `Cmd`）并左键点击圆点，颜色循环切换：
  - ⚪ 默认（半透明灰）
  - 🟢 绿色
  - 🔴 红色
- **缩略导航条同步**：右侧缩略导航条上的小点会同步显示对应消息的标记颜色，快速定位重点内容。
- **状态持久化**：所有标记通过浏览器本地存储（`browser.storage.local` / `chrome.storage.local`）保存，刷新页面后自动恢复。
- **动态加载支持**：借助 `MutationObserver` 监听 DOM 变化，在滚动加载、新建对话、切换会话时自动为新消息注入标签。
- **虚拟列表兼容**：针对 DeepSeek 右侧使用的虚拟列表（DOM 节点复用），采用基于消息文本哈希的稳定 ID，避免标签错位。
- **跨浏览器适配**：内置存储适配层，自动检测 `browser.*` 或 `chrome.*` 命名空间，兼容 Firefox 与 Chromium 系浏览器。
- **轻量无侵入**：所有 UI 元素均通过 CSS 注入，不修改页面原有结构，不影响 DeepSeek 正常功能。

---

## 快速开始

### 1. 环境要求

- Node.js 18+
- npm 9+
- 任意基于 Chromium 或 Gecko 内核的浏览器（Chrome、Edge、Firefox、联想浏览器等）

### 2. 从源码构建

```bash
# 克隆仓库
git clone https://github.com/HU-SHD/deepseek-chat-tagger.git
cd deepseek-chat-tagger

# 安装依赖
npm install

# 构建所有浏览器版本（输出到 dist/ 目录）
npm run build
```

构建完成后，`dist/` 目录下会生成 `chrome/`、`firefox/` 等子目录，分别对应不同浏览器的扩展包。

### 3. 加载到浏览器

**Chrome / Edge / 联想浏览器**：
1. 打开 `chrome://extensions/`（Edge 为 `edge://extensions/`）。
2. 开启右上角「开发者模式」。
3. 点击「加载已解压的扩展程序」。
4. 选择 `dist/chrome/` 目录。

**Firefox**：
1. 打开 `about:debugging#/runtime/this-firefox`。
2. 点击「临时加载附加组件」。
3. 选择 `dist/firefox/manifest.json` 文件。

### 4. 使用方法

1. 打开 [chat.deepseek.com](https://chat.deepseek.com/) 并进入任意对话。
2. 在右侧每条消息的右上角可以看到一个半透明灰点。
3. 按住 `Ctrl`（Mac 为 `Cmd`）并左键点击灰点：
    - 第一次：变绿 🟢
    - 第二次：变红 🔴
    - 第三次：恢复默认 ⚪
4. 右侧缩略导航条上的对应小点会同步变色，方便快速定位。

---

## 快捷键与操作说明

| 操作 | 快捷键 | 说明 |
|------|--------|------|
| 切换标签颜色 | `Ctrl` + 左键点击（Mac：`Cmd` + 左键点击） | 在默认（灰）、绿色、红色之间循环切换 |
| 恢复默认 | 连续点击直至循环回灰色 | 清除该消息的标记记录 |
| 刷新页面 | `F5` 或 `Ctrl+R` | 所有标记从本地存储自动恢复 |
| 切换会话 | 点击左侧历史记录 | 自动为新会话的消息注入标签 |

> **提示**：若点击标签时浏览器默认行为被触发（如在新标签页打开链接），扩展已通过 `preventDefault()` 阻止，不会影响正常操作。

---

## 使用示例

假设你在 DeepSeek 中与 AI 进行了多轮对话，想要标记其中三条重要消息：

1. 找到第一条重要消息，按住 `Ctrl` 点击其右上角灰点 → 变为绿色 🟢。
2. 找到第二条重要消息，按住 `Ctrl` 点击灰点两次 → 变为红色 🔴。
3. 找到第三条重要消息，按住 `Ctrl` 点击灰点一次 → 变为绿色 🟢。
4. 观察右侧缩略导航条，对应位置的小点分别变为绿色、红色、绿色，可快速跳转。
5. 按 `F5` 刷新页面，所有标记颜色保持不变。

---

## 源码构建

### 环境要求

- Node.js 18+
- npm 9+

### 构建步骤

```bash
# 安装依赖
npm install

# 构建所有浏览器版本（输出到 dist/ 目录）
npm run build

# 仅构建 Chrome 版本
npm run build:chrome

# 仅构建 Firefox 版本
npm run build:firefox

# 仅构建 Edge 版本
npm run build:edge
```

构建产物位于 `dist/` 目录下，按浏览器分类存放。

---

## 项目结构

```
deepseek-chat-tagger/
├── src/main/resources/extension/   # 扩展源码
│   ├── manifest.json               # Manifest V3 清单
│   ├── content/                    # Content Scripts
│   │   ├── tag-state.js            # 三态状态机（纯逻辑）
│   │   ├── selector-resolver.js    # 多策略 DOM 选择器解析器
│   │   └── content.js              # 主逻辑（注入、事件、存储）
│   ├── styles/                     # 注入样式
│   │   └── injected.css            # 标签与缩略点样式
│   └── libs/                       # 第三方库
│       └── browser-polyfill.js     # webextension-polyfill
├── scripts/                        # 构建脚本（备用）
│   └── build-crx.js                # CRX3 打包脚本（可选）
├── package.json                    # npm 项目描述与构建脚本
├── pom.xml                         # Maven 项目描述（用于 IDEA 管理）
└── README.md                       # 项目说明文档
```

---

## 核心实现原理

### 1. 三态状态机

`tag-state.js` 中实现了一个纯函数状态机，状态循环顺序为 `default → green → red → default`。该模块不依赖任何浏览器 API，可在任意环境中独立测试。

### 2. 多策略 DOM 选择器

`selector-resolver.js` 针对 DeepSeek 页面中可能变化的类名，提供了多组候选选择器，按优先级依次尝试，直到命中目标元素。同时支持从 `href`、`data-*` 属性中提取稳定的会话 ID。

### 3. 消息 ID 生成

由于右侧对话区使用虚拟列表（DOM 节点会复用），插件通过提取消息文本的前 200 个字符计算哈希值，生成稳定的 `messageId`。只要消息内容不变，ID 就不变，从而保证标签不会因滚动而错位。

### 4. 存储适配层

`content.js` 中封装了 `storageGet` 和 `storageSet`，优先使用 `browser.storage.local`（Firefox），若不存在则回退到 `chrome.storage.local`（Chromium），并用 Promise 包装回调，实现跨浏览器兼容。

### 5. 动态监听

使用 `MutationObserver` 监听 `document.body` 的子节点变化，当检测到新消息节点被添加到 DOM 时，通过 `requestAnimationFrame` 节流触发重新扫描，确保动态加载的消息也能及时获得标签。

### 6. 缩略导航条同步

在扫描消息的同时，收集右侧缩略导航条中的所有小点，按索引与消息列表一一对应，将消息的标记状态（`data-state`）同步到缩略点上，并通过 CSS 属性选择器实现颜色同步。

---

## 许可证

本项目采用 [MIT License](https://opensource.org/licenses/MIT) 许可协议。

---

## 变更日志

### [0.1.0] - 2026-10-14
- 首个发布版本。
- 实现右侧对话消息的三态颜色标签（默认/绿色/红色）。
- 支持 `Ctrl` + 左键循环切换。
- 实现标记状态本地持久化。
- 实现右侧缩略导航条颜色同步。
- 支持动态加载消息自动注入。
- 兼容 Chrome、Edge、Firefox、联想浏览器等。

**© 2026 HU-SHD. 保留所有权利。**
```