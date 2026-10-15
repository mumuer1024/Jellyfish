# Jellyfish Lite（轻量版）

> **[English](./README.md) · [简体中文](./README.zh-CN.md)**

> Jellyfish Lite 是 [Jellyfish 主题](https://github.com/n00bcodr/Jellyfish)（作者 **n00bcodr**，DBAD 许可，见仓库根 [`../LICENSE.md`](../LICENSE.md)）的**性能轻量化变体**：**功能 100% 保留**，削减动效与高开销渲染效果，目标是降低渲染压力、减少卡顿。仓库根目录的原版文件**一律未改动**——本 `lite/` 目录是独立、可完整导入的替代主题。

---

## 1. 简介

Jellyfish Lite 保留原版主题的全部功能（布局、配色、字体、交互、插件适配），只移除或收窄消耗渲染性能的部分：

- **移除**全部 `backdrop-filter` 玻璃拟态模糊 → 半透明纯色背景；
- **移除**常驻/无限动画（含全屏背景滚动）与入场、悬停装饰动画；
- **移除** `will-change` 常驻图层提示与大辉光阴影（≥20px）；
- **收窄** `transition: all`（逐处处理）——必要的状态反馈（菜单、抽屉、弹窗、头部固定）保留；
- **默认新增启用**（第二轮）：详情页按钮文字（`icontext.css`）与彩色家长分级（`ratings.css`，需配合 `scripts/rating.js`）；
- **新增**（第三轮）：Nico Remix Logo 机制（主题色 SVG mask，与配色完全解耦）。

详细的变更清单、静态指标与内部验收记录在审计文档 [`../AUDIT.md`](../AUDIT.md)（中文，内部文档）。本文档聚焦**使用方法**。

> 说明：本文中关于"更流畅"的描述为人工 A/B 观察结论，非仪器化基准测试——详见 [`../AUDIT.md`](../AUDIT.md) §7.5。

## 2. 安装

在 Jellyfin 的 **Dashboard → General → Custom CSS Code**（全局）或 **用户设置 → Display → Custom CSS Code**（单用户）导入**一个 URL**：

```css
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css");
```

> ⚠️ **可用性**：本 Fork 尚未发布——`lite/` 文件尚未推送，**在仓库公开并推送前**上述 CDN URL 会 404。本地测试请用任意静态服务器（如仓库根目录执行 `python3 -m http.server`），导入形如 `http://<主机>:8000/lite/theme.css` 的地址。（jsDelivr 的 `gh` 代理会服务任意**公开**仓库、无需 Release；仓库公开后即可使用。）

请保持仓库目录结构完整——所有内部 `@import`（字体、图标、登录页、默认配色、icontext、彩色分级、Logo remix）均为**相对路径**，随入口文件同源解析。不要把 `lite/theme.css` 单独拷走使用；它需要 `fonts/`、`colors/`、`logos/`、`lite/` 等目录在周围。

唯一的外部依赖是 Druidblack 的元数据图标 CDN，与原版主题一致（未改动）。

## 3. 配色说明与切换

原版 15 套配色**全部原样保留、未改动**（`colors/*.css`，纯 `:root` CSS 变量，无任何效果）：

`aurora` · `banana` · `coal` · `coral` · `forest` · `grass` · `jellyblue` · `jellyflix` · `jellypurple` · `lavender` · `midnight` · `mint` · `ocean` · `peach` · `watermelon`

- **默认配色**：`jellyblue`（由 `lite/theme.css` 中的导入行决定）。
- 切换配色只替换 CSS 变量——**`lite/theme.css` 永不会被重新导入**，已优化的性能始终生效。

### 3.1 全局（服务器）切换

编辑 `lite/theme.css` 中的配色 `@import` 行：

```css
@import url("../colors/ocean.css");   /* 全局默认改为 Ocean */
```

### 3.2 单用户（Display 设置）切换

先导入 `lite/theme.css`，**再**追加一行目标配色（后导入的 `:root` 变量生效）：

```css
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css");
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/colors/peach.css"); /* 该设备用 Peach */
```

（自托管？把 URL 换成你自己的静态服务器路径即可。）

### 3.3 动态切换（脚本）

[`../scripts/theme_selector.js`](../scripts/theme_selector.js) 在用户资料设置里提供配色下拉（含"随机每日主题"）。注意：

- 脚本内置的是**上游** CDN 链接（`n00bcodr/Jellyfish`）——若自托管分叉仓库，请把脚本内的 URL 换成你自己的（`scripts/` 为上游文件，Lite 未改动）；
- 它只加载"纯变量"的配色文件，**永远不会重新导入任何主题样式表**——用下拉切换配色不会引入未经优化的原版 `theme.css`。

## 4. Logo 模式

### Logo 的工作原理

Logo 是 **PNG 图片**，通过每个 `colors/*.css` 定义的 `--logo` 变量加载，出现在三处：头部/页面标题 Logo（`.pageTitleWithDefaultLogo`）、登录页 Logo（`#loginPage h1::before`）、Jellyfin 12+ 顶栏服务器 Logo（MUI toolbar）。详情页条目 Logo 使用条目自身美术图，不受影响。

Lite 增加两层机制：**底层**（原版 PNG，由 `--logo-base-image` 驱动）与**叠加层**（伪元素，以 `--logo-remix-color` 填充、由你的 SVG 作 `mask-image` 裁剪）。变量定义在 `lite/logo-remix.css`，元素规则在 `lite/theme.css` 末尾。

> **没有配置界面**——切换模式 = 编辑 `lite/logo-remix.css` 顶部的三个 `:root` 变量（文件底部提供了三种模式的可复制块）。

### 三种模式

用以下任一组替换 `lite/logo-remix.css` 顶部的三行 `:root` 变量：

**A. Nico Remix（默认）**——纯主题色剪影，原版 PNG 隐藏（无双重 Logo）：

```css
--logo-base-image: none;
--logo-mask-source: url("logos/jellyfish-remixed.svg");
--logo-remix-color: var(--primary-accent-color);
```

**B. 原版 Jellyfish PNG**：

```css
--logo-base-image: url("../logos/jellyfish.png");
--logo-mask-source: none;
--logo-remix-color: transparent;
```

**C. Streamberry PNG**：

```css
--logo-base-image: url("../logos/streamberry.png");
--logo-mask-source: none;
--logo-remix-color: transparent;
```

### 使用你自己的 SVG（Nico Remix）

- 把你的 SVG 放到 **`./logos/jellyfish-remixed.svg`**（即 `lite/logos/jellyfish-remixed.svg`）——**不是**仓库根目录的 `logos/`（那里只放原版 PNG）。自定义属性值中的相对 URL 按"定义该变量的样式表"（即 `lite/`）解析，放错目录会 404。
- 绘制要求：单色剪影、透明背景（mask 只取 alpha 通道——显示颜色始终来自主题）、viewBox 宽高比建议 **2.36:1**（与原版 640×271 一致）、不留透明留白、无外部引用。完整清单见 [`./logos/README.md`](./logos/README.md)。
- SVG 放入后**无需改任何 CSS**——Nico Remix 就是默认模式。

### 回退与逃生口

- CSS 无法检测 mask 是否加载成功（失败的 mask 会隐藏其元素——[w3c/fxtf-drafts #130](https://github.com/w3c/fxtf-drafts/issues/130)），因此"SVG 正常时隐藏 PNG"与"SVG 缺失时自动显示 PNG"**不可兼得**。remix 模式选择正确显示：SVG 缺失时 Logo 区域**为空**（不会显示 PNG）。**手动回退** = 切到 B 或 C 模式（一个块）。
- 若个别浏览器把失败的 mask 渲染成未裁剪的实色矩形，设 `--logo-remix-color: transparent` 消除叠加层（要显示原版 PNG 还需切到 PNG 模式）。
- Logo 与配色**完全解耦**：底层读取 `--logo-base-image`（绝不读取当前配色的 `--logo`）——切换配色（包括 jellyflix）不会改变 Logo。

## 5. 可选 CSS 组件

| 组件 | Lite 文件 | 默认 | 说明 |
|---|---|---|---|
| `theme.css` | `lite/theme.css` | 入口 | 主样式表；自动引用下方各项；内联包含 Jellyfin 12+ dashboard 修复段 |
| `loginpage.css` | `lite/loginpage.css` | 自动 | 登录页样式 |
| `icontext.css` | （原版直接复用） | **启用** | 详情页按钮显示文字（Trailer / Mark-as-watched / Shuffle / More / User-rating）；Play 保留图标 |
| `ratings.css` | `lite/ratings.css` | **启用** | 彩色家长分级（需配合 `scripts/rating.js`） |
| Logo remix | `lite/logo-remix.css` + `lite/logos/` | **启用** | Logo 模式切换与路径变量（见 §4） |
| `12_fixes.css` | `lite/12_fixes.css` | 可选 | Jellyfin 12+ 修复；仅"单独按组件导入"的部署需要（`theme.css` 已内联） |
| `misc.css` | `lite/misc.css` | 可选 | 卡片网格悬停叠层、移动端 OSD 微调 |
| `slideshow.css` | `lite/slideshow.css` | 可选 | 自定义首页 / 幻灯片插件 |
| `progress_bar.css` | `lite/progress_bar.css` | 可选 | 进度条样式 |
| `bg_scroll.css` | `lite/bg_scroll.css` | 可选 | **静态兼容模块**：保留 115% 背景裁切，移除无限滚动动画 |
| `colors/*`、`fonts/*`、`scripts/*`、`indicators.css`、`sidebar.css`、`scrollbar.css`、`card_buttons.css`、`streamberry_logo.css`、`arr-tags-links.css` | （原版直接沿用） | 按需 | 无动效/高开销，原样使用 |

**启用 / 停用**：编辑 `lite/theme.css` 顶部的 `@import` 块（注释掉一行即停用）；或在 Custom CSS 里于 `lite/theme.css` 导入**之后**追加单用户导入（后导入的变量生效）。

## 6. 配套脚本

脚本位于仓库根 [`../scripts/`](../scripts/)（上游文件，未改动），提供 CSS 无法实现的功能。

- **`rating.js`**——`ratings.css` 的配套脚本。Jellyfin 原生不暴露分级信息，该脚本把 `rating` 属性写入 `.mediaInfoOfficialRating`，彩色规则据此生效。**不注入则徽章保持灰色**（`#607D8B`，与原版兜底一致）。用任意 JS 注入方式安装（如 [Jellyfin JavaScript Injector](https://github.com/n00bcodr/Jellyfin-JavaScript-Injector) 插件）。
- **`theme_selector.js`**——资料设置里的配色下拉（见 §3.3）。
- **`title.js`**——把浏览器标签页标题改为 "Jellyfish"。
- **`loginimage.js`**——手动登录页显示用户头像而非用户名（⚠️ 会隐藏用户名输入框——仅当所有用户都可见于登录页时使用）。
- **`dashboard_theming.js`**——Dashboard 主题化（⚠️ 自担风险：Jellyfin 在 10.11 移除了 dashboard 自定义 CSS；需额外插件，见脚本自带说明）。

注入方式与警告详见 [`../scripts/README.md`](../scripts/README.md)。

## 7. 兼容性

- **Jellyfin**：目标版本 **10.11+**，并保留上游主题的 **Jellyfin 12** 样式支持。选择器与上游一致，兼容面与原版主题相同。
- **浏览器**：Logo remix 使用 CSS `mask-image`（含 `-webkit-` 前缀），现代浏览器均支持；§4 的逃生口覆盖个别浏览器渲染差异。
- **减少动态**：保留 `prefers-reduced-motion` 兜底（如分级徽章 hover）。
- **与原版差异（摘要）**：移除模糊、动画、`will-change` 与大辉光阴影；收窄过渡；视觉除动效/模糊外等效。完整变更清单、静态指标与验收记录：[`../AUDIT.md`](../AUDIT.md)。

## 8. 故障排查

| 症状 | 可能原因 / 处理 |
|---|---|
| 主题未生效 | 只保留**一个**入口导入（`lite/theme.css`）；目录结构（`fonts/`、`colors/`、`logos/`、`lite/`）必须完整以支持相对导入。看浏览器 Network 是否有 404，并强刷缓存。 |
| Logo 区域为空（remix 模式） | SVG 缺失或路径错误——必须是 `lite/logos/jellyfish-remixed.svg`，**不是**仓库根 `logos/`。恢复文件，或按 §4 切到 PNG 模式（手动回退）。 |
| Logo 显示实色矩形 | 个别浏览器把失败的 mask 渲染成未裁剪——设 `--logo-remix-color: transparent`（§4）。 |
| 彩色分级显示灰色 | 未注入 `scripts/rating.js`——安装即可（§6）。未注入时灰色属预期兜底。 |
| 切换配色无效 | 单用户覆盖必须在 `lite/theme.css` **之后**导入（§3.2）；动态下拉脚本指向上游 CDN（§3.3）。 |
| 字体 / 图标 404 | 相对导入需要周边目录；自托管时 `fonts/`（woff2）需正确 MIME（`font/woff2`），跨域时配好 CORS 头。 |
| 详情内容区有深色底块 | 半透明色块（`--secondary-background-transparent`，20% 主色）是**原版既有设计**（叠加 Jellyfin 基础样式），Lite 与原版一致，经决定保留。 |
| 还能看到原版特效 | 只导入 `lite/` 的 URL——不要混入根目录的 `theme.css`（未优化版）。 |

## 9. 许可与署名

- 上游：https://github.com/n00bcodr/Jellyfish —— Copyright (C) 2025 **n00bcodr** —— [DBAD 公共许可](../LICENSE.md)（上游文件，未改动）。
- 本 Lite 变体**保留原作者署名与许可证**；每个文件头部均有变体说明。未修改任何上游文件。
- 分发请遵循 DBAD 许可（保留署名；不得改名原样售卖等）。
- 内部文档（变更清单、静态指标、验收记录）：[`../AUDIT.md`](../AUDIT.md)。
