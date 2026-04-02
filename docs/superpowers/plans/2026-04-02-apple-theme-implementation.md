# SitMan Web Apple Theme Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 将 SitMan Web 的视觉风格升级为苹果系浅色玻璃感主题，并支持 4 套预设色板切换与持久化。

**Architecture:** 保持单文件页面结构和现有交互逻辑不变，仅在 `index.html` 内完成样式变量化、主题切换控件和本地持久化逻辑。主题系统通过 CSS 变量和 `body[data-theme]` 驱动，确保移动端样式继续生效。

**Tech Stack:** 原生 HTML、CSS、JavaScript、`localStorage`

---

### Task 1: 建立主题变量与苹果系默认风格

**Files:**
- Modify: `/Users/jimdeng815/projects/sitman-web/index.html`

- [ ] **Step 1: 盘点现有颜色和样式入口**

检查 `index.html` 中的内联 `<style>`，确认以下区域需要切换到变量：

```html
<style>
body { background: #1a1a2e; color: #e0e0e0; }
#titlebar { background: #0f3460; }
#left-panel, #right-panel { background: #16213e; }
#player-bar { background: #0f3460; }
.btn.primary { background: #e94560; }
</style>
```

- [ ] **Step 2: 先写出目标变量结构**

在 `<style>` 顶部增加变量，先不删除旧规则：

```css
:root {
  --bg-start: #eef3f8;
  --bg-end: #f9fbfd;
  --surface: rgba(255, 255, 255, 0.72);
  --surface-strong: rgba(255, 255, 255, 0.92);
  --surface-soft: rgba(255, 255, 255, 0.56);
  --text-primary: #18212f;
  --text-secondary: #617085;
  --accent: #0a84ff;
  --accent-soft: rgba(10, 132, 255, 0.14);
  --border: rgba(107, 127, 153, 0.18);
  --shadow: 0 20px 50px rgba(132, 148, 168, 0.18);
}
```

- [ ] **Step 3: 替换页面基础层级样式**

将 `body`、标题栏、三栏面板、控制条、弹窗、按钮、输入框、滚动条的硬编码颜色替换为变量：

```css
body {
  background:
    radial-gradient(circle at top left, rgba(255,255,255,0.92), transparent 35%),
    linear-gradient(135deg, var(--bg-start), var(--bg-end));
  color: var(--text-primary);
}

#titlebar,
#left-panel,
#right-panel,
#player-bar,
#modal-box,
.note-block {
  background: var(--surface);
  border: 1px solid var(--border);
  box-shadow: var(--shadow);
  backdrop-filter: blur(18px);
}
```

- [ ] **Step 4: 统一强调色和选中态**

将按钮高亮、选中文件、A/B 循环激活态、时间戳链接等统一绑定 `--accent`：

```css
.btn.primary,
.ctrl-btn.active,
.file-item.active {
  background: linear-gradient(135deg, var(--accent), color-mix(in srgb, var(--accent) 70%, white));
  color: white;
  border-color: transparent;
}

.ts-link,
.note-view .ts-link,
#time-display,
#ab-info {
  color: var(--accent);
}
```

- [ ] **Step 5: 目视检查默认主题是否完整**

Run: `rg -n "1a1a2e|0f3460|16213e|e94560|FFD700" /Users/jimdeng815/projects/sitman-web/index.html`
Expected: 仅剩极少数有意保留或注释中的旧色值；主体样式已改用变量。

### Task 2: 增加预设主题切换与持久化

**Files:**
- Modify: `/Users/jimdeng815/projects/sitman-web/index.html`

- [ ] **Step 1: 在标题栏加入主题切换入口**

在标题栏中新增一个轻量主题区：

```html
<div id="theme-switcher" aria-label="主题切换">
  <button class="theme-chip active" data-theme="classic" title="经典蓝"></button>
  <button class="theme-chip" data-theme="mint" title="薄荷绿"></button>
  <button class="theme-chip" data-theme="graphite" title="石墨灰"></button>
  <button class="theme-chip" data-theme="sand" title="暖沙金"></button>
</div>
```

- [ ] **Step 2: 为主题按钮补充样式**

为切换区和按钮增加胶囊式样式：

```css
#theme-switcher {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 6px;
  border-radius: 999px;
  background: var(--surface-soft);
  border: 1px solid var(--border);
}

.theme-chip {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 2px solid rgba(255,255,255,0.85);
  cursor: pointer;
}
```

- [ ] **Step 3: 定义 4 套主题配置**

在脚本中增加主题表：

```js
const THEMES = {
  classic: { accent: "#0a84ff", bgStart: "#eef3f8", bgEnd: "#f9fbfd" },
  mint: { accent: "#34c759", bgStart: "#eef8f3", bgEnd: "#fbfefb" },
  graphite: { accent: "#5e5ce6", bgStart: "#edf0f5", bgEnd: "#f7f8fb" },
  sand: { accent: "#c98b2e", bgStart: "#f8f2e8", bgEnd: "#fcfaf6" }
};
```

- [ ] **Step 4: 写最小主题切换逻辑**

在现有脚本底部增加：

```js
function applyTheme(name) {
  const theme = THEMES[name] || THEMES.classic;
  document.body.dataset.theme = name;
  document.documentElement.style.setProperty("--accent", theme.accent);
  document.documentElement.style.setProperty("--bg-start", theme.bgStart);
  document.documentElement.style.setProperty("--bg-end", theme.bgEnd);
  localStorage.setItem("sitman-theme", name);
}

document.querySelectorAll(".theme-chip").forEach((chip) => {
  chip.addEventListener("click", () => applyTheme(chip.dataset.theme));
});

applyTheme(localStorage.getItem("sitman-theme") || "classic");
```

- [ ] **Step 5: 检查刷新后是否保持主题**

Run: `rg -n "sitman-theme|applyTheme|THEMES|theme-chip" /Users/jimdeng815/projects/sitman-web/index.html`
Expected: 主题配置、切换逻辑和 `localStorage` 持久化代码全部存在。

### Task 3: 修整移动端与可读性细节

**Files:**
- Modify: `/Users/jimdeng815/projects/sitman-web/index.html`

- [ ] **Step 1: 让主题切换在手机端不挤占标题栏**

在已有媒体查询中补充：

```css
@media (max-width: 900px) {
  #theme-switcher {
    order: 4;
    width: 100%;
    justify-content: flex-start;
  }
}
```

- [ ] **Step 2: 调整卡片、按钮、输入框在手机上的留白**

继续在移动端规则中细化：

```css
@media (max-width: 520px) {
  .note-block,
  #left-panel,
  #right-panel,
  #player-bar {
    border-radius: 16px;
  }

  .btn,
  .ctrl-btn,
  #speed-select {
    min-height: 40px;
  }
}
```

- [ ] **Step 3: 检查标题、文件名、状态和主题区的换行**

Run: `sed -n '1,220p' /Users/jimdeng815/projects/sitman-web/index.html`
Expected: 标题栏结构包含 `h1`、文件名、播放状态和主题切换器，且媒体查询中存在换行规则。

### Task 4: 自检与提交准备

**Files:**
- Modify: `/Users/jimdeng815/projects/sitman-web/index.html`
- Review: `/Users/jimdeng815/projects/sitman-web/docs/superpowers/specs/2026-04-02-apple-theme-design.md`

- [ ] **Step 1: 对照设计说明做覆盖检查**

核对以下要求都能在代码里对应到：

```text
1. 苹果系浅色玻璃感
2. 4 套预设色板
3. localStorage 持久化
4. 不改业务逻辑
5. 移动端不退化
```

- [ ] **Step 2: 做静态检查**

Run: `rg -n "#theme-switcher|theme-chip|localStorage|--accent|backdrop-filter|@media \\(max-width: 900px\\)|@media \\(max-width: 520px\\)" /Users/jimdeng815/projects/sitman-web/index.html`
Expected: 主题变量、切换器、持久化和移动端样式都能定位到。

- [ ] **Step 3: 准备提交**

```bash
git -C /Users/jimdeng815/projects/sitman-web add index.html docs/superpowers/specs/2026-04-02-apple-theme-design.md docs/superpowers/plans/2026-04-02-apple-theme-implementation.md
git -C /Users/jimdeng815/projects/sitman-web commit -m "feat: add apple-style themes"
```
