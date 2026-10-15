# Jellyfish Lite（轻量版）

> **[English](./README.md) · [简体中文](./README.zh-CN.md)**

> Jellyfish Lite 是 [Jellyfish 主题](https://github.com/n00bcodr/Jellyfish)（作者 **n00bcodr**，DBAD 许可，见仓库根 [`../LICENSE.md`](../LICENSE.md)）的**性能轻量化变体**：**尽可能保留功能行为**，削减装饰性动效与高开销渲染效果，目标是降低渲染压力、减少卡顿。仓库根目录的原版文件**一律未改动**——本 `lite/` 目录是独立、可完整导入的替代主题。

---

## 1. 简介

Jellyfish Lite 尽可能保留原版主题的功能（布局、配色、字体、交互、插件适配），只移除或收窄消耗渲染性能的装饰性部分：

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

`@main` 为当前推荐的**浮动版本入口**：它始终跟随仓库 `main` 分支的最新内容（每次推送都会更新），**并非固定版本**。`lite/` 文件已随 `main` 分支发布并可正常访问。

> **CDN 用户** —— 这一行 `@import` 即全部所需。主主题、默认配色及已集成样式都通过它加载。下面所有配置（配色切换、Logo 模式、可选组件）都只需在同一个 **自定义 CSS 代码** 框中追加内容，无需访问仓库。
>
> **自行托管用户** —— 把仓库克隆/复制到自己的静态服务器，从自己的域名导入 `lite/theme.css`。请保持目录结构完整——所有内部 `@import`（字体、图标、登录页、默认配色、icontext、彩色分级、Logo remix）均为**相对路径**，随入口文件同源解析。不要把 `lite/theme.css` 单独拷走使用；它需要 `fonts/`、`colors/`、`logos/`、`lite/` 等目录在周围。

> ⚠️ **版本固定说明：** 仓库已有一个旧 Tag `1.0`，但它早于 Jellyfish Lite，**不包含 `lite/` 文件**——请勿使用 `@1.0`（其 `lite/*` 路径会 404）。在发布 Lite 专属 Tag 之前，请继续使用 `@main`。

唯一的外部依赖是 Druidblack 的元数据图标 CDN，与原版主题一致（未改动）。

## 3. 配色说明与切换

原版 15 套配色**全部原样保留、未改动**（`colors/*.css`，以纯 `:root` CSS 变量为主，个别配色还引用背景图 URL）：

`aurora` · `banana` · `coal` · `coral` · `forest` · `grass` · `jellyblue` · `jellyflix` · `jellypurple` · `lavender` · `midnight` · `mint` · `ocean` · `peach` · `watermelon`

- **默认配色**：`jellyblue`（由 `lite/theme.css` 中的导入行决定）。
- 切换配色只替换 CSS 变量——**`lite/theme.css` 永不会被重新导入**，已优化的性能始终生效。

### 3.1 推荐：在自定义 CSS 中切换配色（所有用户通用）

先导入 `lite/theme.css`，**再**追加一行目标配色——后导入的 `:root` 变量生效。此方法在 **控制台 → 常规 → 自定义 CSS 代码**（全局）与 **用户设置 → Display → 自定义 CSS 代码**（单用户）中都适用，无需访问仓库：

```css
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css");
@import url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/colors/peach.css"); /* 本服务器/设备用 Peach */
```

（自行托管？把 URL 换成你自己的静态服务器路径即可。）

### 3.2 仅自行托管：修改全局默认配色

自行托管仓库时，也可以直接编辑 `lite/theme.css` 中的配色 `@import` 行来改变全局默认配色：

```css
@import url("../colors/ocean.css");   /* 全局默认改为 Ocean */
```

### 3.3 动态切换（脚本）

[`../scripts/theme_selector.js`](../scripts/theme_selector.js) 在用户资料设置里提供配色下拉（含"随机每日主题"）。注意：

- 脚本内置的是**上游** CDN 链接（`n00bcodr/Jellyfish`）——若自托管分叉仓库，请把脚本内的 URL 换成你自己的（`scripts/` 为上游文件，Lite 未改动）；
- 它只加载"纯变量"的配色文件，**永远不会重新导入任何主题样式表**——用下拉切换配色不会引入未经优化的原版 `theme.css`。

## 4. Logo 模式

### Logo 的工作原理

Logo 是 **PNG 图片**，通过每个 `colors/*.css` 定义的 `--logo` 变量加载，出现在三处：头部/页面标题 Logo（`.pageTitleWithDefaultLogo`）、登录页 Logo（`#loginPage h1::before`）、Jellyfin 12+ 顶栏服务器 Logo（MUI toolbar）。详情页条目 Logo 使用条目自身美术图，不受影响。

Lite 增加两层机制：**底层**（原版 PNG，由 `--logo-base-image` 驱动）与**叠加层**（伪元素，以 `--logo-remix-color` 填充、由你的 SVG 作 `mask-image` 裁剪）。变量定义在 `lite/logo-remix.css`，元素规则在 `lite/theme.css` 末尾。

### 切换模式——用户级 CSS 覆盖（推荐，所有用户通用）

**没有配置界面**。Logo 由三个 `:root` 变量控制，你可以在同一个 **自定义 CSS 代码** 框中覆盖它们——这是推荐方法，也是 CDN 用户唯一可行的方法（`lite/` 文件在 CDN 上，无法编辑）。

> ⚠️ **在你自己的 Custom CSS 中请使用绝对 HTTPS URL。** 你写在 Jellyfin 自定义 CSS 里的相对 `url()` 会相对 **Jellyfin 页面 URL** 解析（而非 CDN 样式表），几乎必然 404。请直接复制下方任一代码块。

**A. Nico Remix（默认）**——纯主题色剪影，原版 PNG 隐藏（无双重 Logo）：

```css
:root {
  --logo-base-image: none;
  --logo-mask-source: url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/logos/jellyfish-remixed.svg");
  --logo-remix-color: var(--primary-accent-color);
}
```

**B. 原版 Jellyfish PNG**：

```css
:root {
  --logo-base-image: url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/logos/jellyfish.png");
  --logo-mask-source: none;
  --logo-remix-color: transparent;
}
```

**C. Streamberry PNG**：

```css
:root {
  --logo-base-image: url("https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/logos/streamberry.png");
  --logo-mask-source: none;
  --logo-remix-color: transparent;
}
```

Nico Remix 即使不做任何覆盖也是默认模式，所以大多数 CDN 用户**无需**任何 Logo 覆盖。

**自行托管替代方案：** 自行托管仓库时，也可以直接编辑 `lite/logo-remix.css` 顶部的三行 `:root` 变量（文件底部提供了可复制块）。此时相对路径（如 `url("../logos/jellyfish.png")`）可用，因为 `logo-remix.css` 与 `theme.css` 同处 `lite/` 目录（见下）。

### 使用你自己的 SVG（Nico Remix）

- **自行托管：** 把 SVG 放到 **`./logos/jellyfish-remixed.svg`**（即 `lite/logos/jellyfish-remixed.svg`）——**不是**仓库根目录的 `logos/`（那里只放原版 PNG）。SVG 放入后**无需改任何 CSS**——Nico Remix 就是默认模式。
- **CDN 用户：** 无法向 CDN 副本添加文件。请把 SVG 托管到任意公开位置（你自己的服务器、其它 CDN、GitHub Gist 等），再把 `--logo-mask-source` 指向其绝对 `https://` URL（上方 A 代码块，替换 URL 即可）。
- 绘制要求：单色剪影、透明背景（mask 只取 alpha 通道——显示颜色始终来自主题）、viewBox 宽高比建议 **2.36:1**（与原版 640×271 一致）、不留透明留白、无外部引用。完整清单见 [`./logos/README.md`](./logos/README.md)。
- 路径说明：自定义属性中的相对 `url()` 在**使用**该变量时（经 `var()` 替换）才解析，基准是**使用方**样式表（`lite/theme.css`），而非声明方。本主题中两文件同处 `lite/` 目录，故两种基准结果相同；但你在**自己的** Custom CSS 里写的相对路径会相对 Jellyfin 页面解析并 404——请用绝对 URL（见上）。

### 回退与逃生口

- **mask URL 404（SVG 缺失）：** 加载失败的 `mask-image` 会隐藏其作用元素（当前 CSS Masking 行为——[w3c/fxtf-drafts #130](https://github.com/w3c/fxtf-drafts/issues/130)）。纯 CSS 无法检测 mask 是否加载成功，因此"SVG 正常时隐藏 PNG"与"SVG 缺失时自动显示 PNG"**不可兼得**。remix 模式选择正确显示：SVG 缺失时 Logo 区域**为空**（不会显示 PNG）。**手动回退** = 切到 B 或 C 模式（一个块）。
- **mask 为 `none`（未应用 SVG）：** 若 `--logo-mask-source` 计算为 `none`（未设置或显式设为 none）而 `--logo-remix-color` 为不透明色，叠加层**无遮罩**，会渲染成覆盖整个 Logo 区域的**未裁剪实色矩形**。这是标准行为，并非浏览器怪癖。解决办法：设 `--logo-remix-color: transparent`，**并**切到 PNG 模式（B 或 C）。
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

**启用 / 停用**：
- **自行托管：** 编辑 `lite/theme.css` 顶部的 `@import` 块，注释掉一行即停用对应组件（`@import` 必须保持在文件顶部）。
- **CDN 用户：** `lite/theme.css` 已内置的组件（loginpage、icontext、ratings、Logo remix）随入口导入加载，无法在 Custom CSS 中"反导入"。如需改变其表现，请在主题导入**之后**追加更高优先级的覆盖（后置规则 / `!important` 生效）。可选独立文件（`misc.css`、`slideshow.css`、`progress_bar.css`、`bg_scroll.css`、`12_fixes.css`）需要时才单独导入。

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
- **浏览器**：Logo remix 使用 CSS `mask-image`（含 `-webkit-` 前缀），现代浏览器均支持；§4 覆盖了两种失败情形（SVG 404 时区域为空、mask 为 `none` 时实色矩形）。
- **减少动态**：保留 `prefers-reduced-motion` 兜底（如分级徽章 hover）。
- **与原版差异（摘要）**：移除模糊、动画、`will-change` 与大辉光阴影；收窄过渡；视觉除动效/模糊外等效。完整变更清单、静态指标与验收记录：[`../AUDIT.md`](../AUDIT.md)。

## 8. 故障排查

| 症状 | 可能原因 / 处理 |
|---|---|
| 主题未生效 | 检查导入地址是否正好是 `https://cdn.jsdelivr.net/gh/mumuer1024/Jellyfish@main/lite/theme.css`，并只保留**一个**入口导入。自行托管时，目录结构（`fonts/`、`colors/`、`logos/`、`lite/`）必须完整以支持相对导入。看浏览器 Network 是否有 404，并强刷缓存。 |
| Logo 区域为空（remix 模式） | mask SVG 缺失或其 URL 404。默认安装下必须是 `lite/logos/jellyfish-remixed.svg`（**不是**仓库根 `logos/`）。若你在 Custom CSS 里用相对 URL 覆盖了 `--logo-mask-source`，那会必然 404——请用绝对 URL（§4）。也可切到 PNG 模式作为手动回退。 |
| Logo 显示实色矩形 | `--logo-mask-source` 计算为 `none` 而 `--logo-remix-color` 为不透明色——叠加层未遮罩。设 `--logo-remix-color: transparent` 并切到 PNG 模式（§4）。 |
| 彩色分级显示灰色 | 未注入 `scripts/rating.js`——安装即可（§6）。未注入时灰色属预期兜底。 |
| 切换配色无效 | 覆盖必须在 `lite/theme.css` **之后**导入（§3.1）；动态下拉脚本指向上游 CDN（§3.3）。 |
| 字体 / 图标 404 | 相对导入需要周边目录；自托管时 `fonts/`（woff2）需正确 MIME（`font/woff2`），跨域时配好 CORS 头。 |
| 详情内容区有深色底块 | 半透明色块（`--secondary-background-transparent`，20% 主色）是**原版既有设计**（叠加 Jellyfin 基础样式），Lite 与原版一致，经决定保留。 |
| 还能看到原版特效 | 只导入 `lite/` 的 URL——不要混入根目录的 `theme.css`（未优化版）。 |

## 9. 许可与署名

- 上游：https://github.com/n00bcodr/Jellyfish —— Copyright (C) 2025 **n00bcodr** —— [DBAD 公共许可](../LICENSE.md)（上游文件，未改动）。
- 本 Lite 变体**保留原作者署名与许可证**；每个文件头部均有变体说明。未修改任何上游文件。
- 分发请遵循 DBAD 许可（保留署名；不得改名原样售卖等）。
- 内部文档（变更清单、静态指标、验收记录）：[`../AUDIT.md`](../AUDIT.md)。
