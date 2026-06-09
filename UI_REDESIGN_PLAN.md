# 🎨 go-music-dl UI 改造计划：Apple Music for macOS

> 设计目标：以 Apple Music for macOS 为参考基准，将当前 web 界面全面改造为符合 Apple 设计语言的专业音乐应用，删除看板娘小人，彻底重构配色、布局、交互动效。

---

## P0 · 必须先做（破坏性/阻塞性）

### 1. 移除 Sakana 看板娘小人
- **文件**：`pages/index.html`，`static/css/style.css`
- 删除 sakana-widget CSS/JS 的 CDN 引用（`sakana.min.css`、`sakana.min.js`）
- 删除 `<div id="sakana-container">` DOM 节点
- 删除 `initSakanaWidget()` 函数
- 删除 `.sakana-container` 及其子元素所有 CSS
- 删除 .sakana-btn 样式

---

## P1 · 设计 Token 体系（CSS 变量全面重构）

### 2. 建立 Apple Music 设计 Token（:root CSS 变量）
**文件**：`static/css/style.css`

替换当前杂乱的硬编码色值，建立统一 Token 层：

```css
:root {
  /* Apple Music 核心色板 */
  --color-red:     #fa586a;
  --color-pink:    #fc375b;
  --color-blue:    #0a84ff;
  --color-teal:    #5ac8fa;
  --color-green:   #30d158;
  --color-orange:  #ff9f0a;

  /* 昼夜模式（先做浅色） */
  --bg-primary:        #f5f5f7;
  --bg-secondary:      #ffffff;
  --bg-tertiary:       #f2f2f7;
  --bg-glass:          rgba(255, 255, 255, 0.72);

  --text-primary:      #1d1d1f;
  --text-secondary:    #6e6e73;
  --text-tertiary:     #aeaeb2;
  --text-inverse:      #ffffff;

  --border-light:      rgba(0,0,0,0.06);
  --border-medium:     rgba(0,0,0,0.12);
  --border-vibrant:    rgba(0,0,0,0.18);

  --shadow-sm:         0 1px 3px rgba(0,0,0,0.04);
  --shadow-md:         0 4px 14px rgba(0,0,0,0.06);
  --shadow-lg:         0 8px 30px rgba(0,0,0,0.08);

  --radius-sm:  8px;
  --radius-md:  14px;
  --radius-lg:  20px;
  --radius-xl:  28px;
  --radius-full: 999px;

  --font-display: -apple-system, "SF Pro Display", "Helvetica Neue", sans-serif;
  --font-body:    -apple-system, "SF Pro Text", "Helvetica Neue", sans-serif;
  --font-mono:    "SF Mono", "Menlo", monospace;

  --transition-fast: 0.15s ease;
  --transition-smooth: 0.3s cubic-bezier(0.25, 0.1, 0.25, 1);
  --transition-spring: 0.4s cubic-bezier(0.22, 0.61, 0.36, 1);
}
```

---

## P2 · 全局布局重构

### 3. 移除暗色渐变背景，改用 Apple 纯浅灰背景
**文件**：`static/css/style.css`

- 删除 `body` 上的 `radial-gradient` 三层渐变背景
- 替换为 `background: var(--bg-primary)`（#f5f5f7）
- 删除五彩斑斓的功能按钮渐变（`.btn-pill-fav/category/mine` 的 pink/blue/teal 渐变）
- 统一为 Apple 风格的简洁半透明/灰阶按钮

### 4. 标题区重新设计
**文件**：`partials/search_box.html`

- 移除 `<h1>` 中的 🎵 emoji
- 品牌名改为纯文本 `Music`，去除 GitHub 外链
- 字体使用 `var(--font-display)`，字重 700，`font-size: 34px`，`letter-spacing: -0.5px`
- 颜色使用 `var(--text-primary)` 黑色，不再使用渐变色
- 在左侧加入一个圆角方形背景的品牌图标（模拟 app icon）

### 5. 搜索类型切换器改造
**文件**：`partials/search_box.html`，`static/css/style.css`

- 从 radio + label 改为 Apple 风格的 segmented control（磨砂胶囊）
```html
<div class="segmented-control">
  <button class="segment active">歌曲</button>
  <button class="segment">歌单</button>
  <button class="segment">专辑</button>
</div>
```
- 背景 `var(--bg-tertiary)`，选中项白色圆角胶囊（`var(--bg-secondary)` + `shadow-sm`）
- 字体大小 14px，字重 500/600
- 动画：选中指示器滑动过渡

### 6. 搜索框 Apple 化
**文件**：`partials/search_box.html`，`static/css/style.css`

- 搜索框：圆角 `--radius-full`，背景 `var(--bg-tertiary)`，边框无
- placeholder 颜色 `var(--text-tertiary)`
- focus 时出现蓝色环形光圈（`box-shadow: 0 0 0 3px rgba(10,132,255,0.25)`）
- 搜索按钮：从绿色渐变改为深灰/黑色圆角按钮（`var(--text-primary)` 背景），内嵌白色放大镜图标
- 按钮 hover 放大 + 加深

### 7. 功能按钮行精简
**文件**：`partials/search_box.html`，`static/css/style.css`

- 将所有五彩按钮统一为 Apple 风格「药丸」按钮
  - 默认：`bg: var(--bg-tertiary)`，文字 `var(--text-secondary)`
  - hover：`bg: var(--border-medium)`
  - 仅"每日推荐"用红色强调（Apple Music 红）
- 图标从 FontAwesome 改 Apple SF Symbols 风格（保持 FA 但减少花哨感）

---

## P3 · 歌曲卡片重构

### 8. 歌曲列表卡片 Apple Music 化
**文件**：`partials/song_list.html`，`static/css/style.css`

参考 Apple Music 曲目列表设计：
- 卡片移除圆角卡片形态，改为**表格行布局**（Hover 时整行高亮）
  ```
  [封面 48x48 圆角8px] [歌名/歌手] [时长] [来源tag] [操作按钮]
  ```
- 删除 `.song-card` 的白色卡片阴影样式
- 改为：
  - 默认背景透明
  - hover 背景 `var(--bg-tertiary)`（浅灰高亮）
  - 选中/播放中背景 `rgba(10,132,255,0.08)`（浅蓝高亮）带左侧蓝色指示条
- 封面：从 64px 缩小到 48px，圆角 `--radius-sm`（8px）
- 歌名：15px 字重 600，文本溢出省略号
- 歌手：13px `--text-secondary`，可点击链接
- 时长：12px `--text-tertiary`，等宽数字字体
- 来源 tag：小号圆角背景标签

### 9. 操作按钮行重构
**文件**：`partials/song_list.html`，`static/css/style.css`

- 操作按钮默认隐藏，hover 卡片时才滑入显示
- 按钮从彩色圆形改为：32x32px 圆角 8px，`background: var(--bg-tertiary)`，图标 16px
- hover 时背景加深
- 主要操作（播放/下载）保留颜色区分：播放=蓝色，下载=绿色
- 删除按钮 hover 变红色

---

## P4 · 弹窗/模态框 Apple 化

### 10. 设置弹窗改造
**文件**：`partials/modals.html`，`static/css/style.css`

- 弹窗从白色卡片改为 Apple 风格的**半透明材质面板**
  ```css
  background: rgba(255,255,255,0.92);
  backdrop-filter: blur(40px) saturate(180%);
  ```
- 圆角 `--radius-xl`（28px）
- Header 字体更大更粗
- 关闭按钮改为圆形模糊按钮
- 表单元素：
  - input：方形圆角 10px，浅灰背景，聚焦蓝环
  - toggle switch 改为 iOS 风格（圆形滑块 + 绿色激活态）
  - 按钮统一灰色/蓝色

### 11. 扫码登录弹窗
**文件**：`partials/modals.html`，`static/css/style.css`

- 二维码展示区改为纯白底色，细边框
- 去除当前花哨的绿色放射渐变边框
- 状态提示文字统一样式

---

## P5 · 播放器/底部栏

### 12. APlayer 播放器调整
**文件**：`static/css/style.css`

- 如 APlayer 可配置，改为迷你模式吸附底部
- 背景：`var(--bg-glass)` + `backdrop-filter: blur(40px)` 毛玻璃
- 边框：顶部 0.5px 细线
- 歌词展示简化，去除绿色描边效果

---

## P6 · 细节与动效

### 13. 全局交互动效统一
**文件**：`static/css/style.css`

- 所有 hover 过渡使用 `var(--transition-fast)` 或 `var(--transition-smooth)`
- 删除过于浮夸的 `translateY(-3px)` 卡片弹跳，改用 1px 微移 + 阴影变化
- 按钮/卡片缩放从 `scale(1.1)` 改为 `scale(1.02)`
- 页面滚动条美化（webkit 细滚条）
- 选中状态使用 `transition` 背景色过渡

### 14. 图标替换
**文件**：所有 html partials

- 评估：FontAwesome 6 CDN 当前可用，暂保留
- 但删除装饰性过强的图标（如齿轮设置按钮的旋转动画）

### 15. 返回顶部按钮
**文件**：`static/css/style.css`

- 保持功能，改为 Apple 风格：40x40 白色圆角方形，带阴影，仅图标无文字

### 16. 搜索源选择器（Source Grid）改造
**文件**：`partials/search_box.html`，`static/css/style.css`

- 从彩色边框卡片改为 Apple 风格 Tag/Chip
- 未选中：浅灰背景，灰色文字
- 选中：蓝色背景 + 白色文字
- 折叠/展开动画保留

---

## P7 · 移动端适配修补

### 17. 移动端样式审计
**文件**：`static/css/style.css`

- 确保新设计在 768px 以下不崩
- 卡片在移动端改为列表行布局
- 搜索框全宽
- 功能按钮堆叠或横向滚动

---

## 执行顺序

```
P0 (1项)  →  先删小人，不然后面改的 CSS 都有它的残留
P1 (1项)  →  建立 Token 体系，后续所有改动基于 Token
P2 (5项)  →  全局布局、标题、搜索、按钮 —— 视觉主框架
P3 (2项)  →  歌曲卡片、操作按钮 —— 核心内容区
P4 (2项)  →  弹窗模态框
P5 (1项)  →  播放器
P6 (4项)  →  细节动效
P7 (1项)  →  移动端修补

总计：17 项任务
```
