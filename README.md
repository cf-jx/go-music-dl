# Go Music DL — 聚合音乐搜索与下载

> 🎵 多平台聚合音乐搜索、试听、下载工具。支持 Web 界面、TUI 终端和原生桌面应用。

本仓库基于 [guohuiyuan/go-music-dl](https://github.com/guohuiyuan/go-music-dl) 进行二次开发，衷心感谢原作者 [@guohuiyuan](https://github.com/guohuiyuan) 及所有贡献者为这个项目奠定的坚实基础。本文档仅在原项目基础上补充本分支的改动说明。

---

## 🍎 本分支特色：Apple Music for macOS 风格 UI

我们在原版基础上对前端界面进行了全面重构，采用 **Apple Music for macOS 的视觉语言和交互模式**：

### 布局重构

| 改造项 | 原版 | 现版 |
|--------|------|------|
| 全局布局 | 居中单列容器（900px） | 侧边栏 + 内容区 + 底部播放栏 三栏全屏布局 |
| 侧边栏 | 无 | 248px 毛玻璃侧边栏，内置搜索、导航、分类入口 |
| 搜索区 | 绿色渐变标题、radio 切换 | Segmented Control（分段控件）切换歌曲/歌单/专辑 |
| 歌曲卡片 | 白色圆角阴影卡片 | Apple Music 表格行样式，封面 42px，操作按钮 hover 滑入 |
| 配色方案 | 翠绿色 `#10b981` 为主色 | 苹果蓝 `#0a84ff`，浅灰底色 `#f5f5f7` |
| 设计语言 | 暗色渐变 + 彩色按钮 | 毛玻璃质感 + 克制微动效 + SF Pro 字体 |
| 看板娘 | Sakana 动画小人 | 已移除 |

### 设计 Token 体系

建立了完整的 CSS 变量层，便于后续定制和暗色模式扩展：

- **Apple 色板**：红 `#ff375f`、蓝 `#0a84ff`、绿 `#30d158`、橙 `#ff9f0a`
- **表面层级**：窗口底、侧边栏、内容区、毛玻璃半透明
- **排版**：SF Pro Display（标题）/ SF Pro Text（正文）
- **圆角系统**：6px / 8px / 12px / 16px / 20px / 无限圆角
- **过渡曲线**：标准 ease、ease-in、ease-out

---

## 🚀 快速开始

### 环境要求

- Go 1.25+
- CLI 编译无需其他依赖

### 编译与运行

```bash
# 编译 CLI / Web 服务
go build -ldflags="-s -w" -o music-dl ./cmd/music-dl

# 编译原生桌面版（macOS / Linux）
go build -o music-dl-desktop ./desktop_go/

# 启动 Web 模式
./music-dl web

# 启动桌面版
./music-dl-desktop
```

Web 模式默认监听 `http://localhost:8080/music/`。
桌面版默认监听 `http://127.0.0.1:37777/music/`，并自动打开原生窗口。

### macOS 桌面图标

已在桌面创建 `Music Download.app`，双击即可启动原生桌面应用。如需重建：

```bash
# .app 路径指向 go-music-dl/music-dl-desktop，重新编译后无需重建图标
APP="/Users/$(whoami)/Desktop/Music Download.app/Contents/MacOS"
mkdir -p "$APP"
cat > "$APP/launcher" << 'EOF'
#!/bin/bash
cd "/Users/$(whoami)/workspace/music download/go-music-dl"
exec "./music-dl-desktop"
EOF
chmod +x "$APP/launcher"
```

---

## 🎨 UI 改造清单

详细改造计划见 [UI_REDESIGN_PLAN.md](UI_REDESIGN_PLAN.md)。

### 涉及文件

| 文件 | 改动 |
|------|------|
| `internal/web/templates/pages/index.html` | 侧边栏 + 内容区 + 播放栏三段布局 |
| `internal/web/templates/pages/auth.html` | 品牌标识简化 |
| `internal/web/templates/partials/search_box.html` | 分段控件 + 极简搜索框 |
| `internal/web/templates/partials/song_list.html` | 表格行样式 + hover 操作按钮 |
| `internal/web/templates/static/css/style.css` | 完全重写：Token 层 + 布局系统 + 组件样式 |

---

## 📖 原项目功能（保持完整）

以下功能均来自原项目，本分支未做删减，仅重构了视觉层：

- 13 个音乐平台搜索支持（网易云、QQ、酷狗、酷我、咪咕、千千、汽水、5sing、Jamendo、JOOX、Bilibili、Apple Music）
- FLAC 无损下载
- 逐字歌词（原文/译文/罗马音）与卡拉 OK 高亮
- 歌单/专辑搜索与批量处理
- 本地歌单管理
- 本地音乐管理（扫描、上传、元数据补全）
- Cookie 扫码登录
- 视频渲染导出
- TUI 终端模式
- Docker 部署
- GitHub Actions 自动构建
- Android APK 构建支持

---

## 🏗️ 项目结构

```
go-music-dl/
├── cmd/music-dl/              # CLI / TUI 主程序入口
├── core/                      # 核心业务逻辑（搜索、下载、配置）
├── internal/
│   ├── appshell/              # 桌面壳服务启动层
│   ├── cli/                   # TUI 界面（Bubble Tea）
│   └── web/                   # Web 后端（Gin）+ 前端模板
│       ├── templates/         # HTML 模板与静态资源
│       │   ├── pages/         # 页面模板
│       │   ├── partials/      # 组件模板
│       │   └── static/        # CSS / JS
│       ├── server.go          # Web 服务入口
│       ├── music.go           # 搜索与解析路由
│       ├── collection.go      # 本地歌单接口
│       └── local_music.go     # 本地音乐管理
├── desktop/                   # Rust 桌面壳（Tao + Wry）
├── desktop_go/                # Go 桌面壳（WebView，本分支推荐）
├── desktop_app/               # 移动端（Gio）
└── data/                      # 运行时数据（下载、配置、数据库）
```

---

## 🤝 致谢

本仓库是对 [guohuiyuan/go-music-dl](https://github.com/guohuiyuan/go-music-dl) 的 fork 与二次开发。

**特别感谢原作者 [@guohuiyuan](https://github.com/guohuiyuan)** 以及以下间接依赖的优秀开源项目：

- [0xHJK/music-dl](https://github.com/0xHJK/music-dl) — 音乐下载库
- [CharlesPikachu/musicdl](https://github.com/CharlesPikachu/musicdl) — Python 版音乐下载
- [metowolf/Meting](https://github.com/metowolf/Meting) — 多平台音乐聚合接口设计参考
- [Suxiaoqinx/Netease_url](https://github.com/Suxiaoqinx/Netease_url) — 网易云 FLAC 解析
- [Suxiaoqinx/qqmusic_flac](https://github.com/Suxiaoqinx/qqmusic_flac) — QQ 音乐 FLAC 解析
- [chenmozhijin/LDDC](https://github.com/chenmozhijin/LDDC) — 逐字歌词展示思路参考
- [guohuiyuan/music-lib](https://github.com/guohuiyuan/music-lib) — 核心音乐库

---

## 📄 许可证

本项目遵循上游仓库的 [GNU Affero General Public License v3.0 (AGPL-3.0)](LICENSE)。

## ⚠️ 免责声明

仅供学习和技术交流使用。下载的音乐资源请在 24 小时内删除。
