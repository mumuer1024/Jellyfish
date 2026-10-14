# Jellyfish Lite — 轻量版说明

> 本目录是 [Jellyfish 主题](https://github.com/n00bcodr/Jellyfish)（作者 **n00bcodr**，DBAD 许可，见仓库根目录 `LICENSE.md`）的**性能轻量化变体**：**功能 100% 保留，削减动效与高开销渲染效果**，目标是降低渲染压力、减少卡顿。
>
> 仓库根目录的原版文件**一律未改动**，本目录是独立、可完整导入的替代主题。

---

## 一、安装（唯一入口 URL）

只需在 Jellyfin 的 **Dashboard → General → Custom CSS Code**（全局）或 **用户设置 → Display → Custom CSS Code**（单用户）导入**一个 URL**：

```css
@import url("https://cdn.jsdelivr.net/gh/<你的仓库>/Jellyfish/lite/theme.css");
```

- 所有内部 `@import`（字体、图标、登录页、默认配色、icontext、彩色分级）已改为**相对路径**，随入口文件同源解析；只需保持 `fonts/`、`colors/`、`logos/` 等目录结构与原仓库一致即可。
- 若自托管，用对应静态 URL 替换上面占位即可。
- 外部依赖仅保留 Druidblack 的元数据图标 CDN（原主题同样依赖，未改动）。
- **默认已启用**：详情页按钮文字（原版 `icontext.css`，规则直接复用）与彩色家长分级（`ratings.css` Lite 版）。彩色分级需配合 `scripts/rating.js` 才会着色，详见「三、变更清单」与「七、人工验收结果」。
- **Logo 二次开发（默认启用、安全回退）**：Logo 为 PNG 图片，通过 `--logo` 变量加载；Lite 用 CSS `mask-image` 方案把用户自绘 SVG（`lite/logos/jellyfish-remixed.svg`）以主题色叠加显示。SVG 尚未提供时自动回退显示原版 PNG（不会空白），详见「四、Logo 二次开发」。

## 二、文件对应关系

| 原版文件 | Lite 对应 | 说明 |
|---|---|---|
| `theme.css` | `lite/theme.css` | **主入口**（含 12_fixes 合并段与 dashboard 段），自动引用下方 loginpage / icontext / ratings |
| `loginpage.css` | `lite/loginpage.css` | 已被 `lite/theme.css` 自动引用，无需单独导入 |
| `icontext.css` | 原版文件直接复用 | **已默认导入**（详情页按钮文字），规则未重写 |
| `ratings.css` | `lite/ratings.css` | **已默认导入**（彩色分级，需配合 `scripts/rating.js` 才着色） |
| `12_fixes.css` | `lite/12_fixes.css` | 仅在你想单独导入 12+ 修复时使用 lite 版 |
| `misc.css` | `lite/misc.css` | 可选，用 lite 版替换导入即可 |
| `slideshow.css` | `lite/slideshow.css` | 可选（配合自定义首页/幻灯片插件） |
| `progress_bar.css` | `lite/progress_bar.css` | 可选 |
| `bg_scroll.css` | `lite/bg_scroll.css` | **静态兼容模块**：保留 115% 背景裁切，移除无限滚动动画 |
| —（新增） | `lite/logo-remix.css` + `lite/logos/` | **默认启用**：Logo 模式切换与路径变量（Nico Remix / 原版 Jellyfish / Streamberry），SVG 缺失时自动回退原版 PNG（见「四」） |
| `colors/*`、`fonts/*`、`scripts/*`、`indicators.css`、`sidebar.css`、`scrollbar.css`、`card_buttons.css`、`streamberry_logo.css`、`arr-tags-links.css` | **直接沿用原版** | 这些文件本身无动效/无高开销效果 |

## 三、变更清单

按"等价视觉轻量化"原则，仅移除/收窄动效与渲染开销，**不涉及任何布局、配色、字体、交互行为、插件适配或响应式改动**。

### 全局
- **移除全部 `backdrop-filter` 玻璃拟态模糊**（原版 35 处，含 `blur(50px)`×6、`blur(30px)`×3、`blur(100px)`、`blur(15px)` 默认等多处大面积元素）：改为原有半透明纯色背景（如 `var(--secondary-background-transparent)`）。保留 4 处 `backdrop-filter: none/unset`（用于抵消 Jellyfin 默认样式）与 1 处 `@supports` 特性探测，不影响功能。
- **移除常驻动画**：`bg_scroll.css` 的 `backgroundScroll 50s linear infinite`（全屏背景逐帧重绘）→ 静态展示。
- **移除全部 `will-change`**（2 处，登录页）：不再为静态/悬停元素预留常驻图层。
- **移除大辉光阴影（≥20px）**：登录卡片 50px 辉光、skip 按钮 30px 辉光等（保留常规小投影与功能性阴影）。
- **`transition: all` 与裸缩写逐处清理**：按"菜单、弹窗、头部固定等必要状态反馈保留，纯装饰过渡移除"处理（明细见各文件）。

### `lite/theme.css`
- 移除 24 处 backdrop-filter 模糊声明。
- 移除详情页动效：logo 滑入（0.8s）、主容器淡入滑入（1.5s）、星级 hover 旋转（1s）、评分 hover 倾斜、flat/link 按钮 hover 放大。
- 移除移动端背景图淡入动画（0.8s）。
- 过渡收窄：侧边栏列表项、Up Next 按钮、MUI 侧栏/抽屉项、fab/raised、引导频道图片（filter 过渡删除）、日期标签页等改为具体廉价属性（color/background/opacity/transform）。
- 保留：菜单/抽屉滑入（transform/left）、头部固定过渡（`.5s`）、进度条标记 opacity 反馈、alpha 选择器反馈等。
- 导入路径改为相对路径。

### `lite/loginpage.css`
- 全屏背景 `filter: blur(5px) brightness(0.5)` → `filter: brightness(0.5)`（去全屏模糊，保留压暗以保证文字可读）。
- 移除登录淡入动画与 `will-change`。
- 移除用户卡片 hover 缩放+辉光（保留 `:focus-within` 轮廓，无障碍不受影响）。
- 对话框/输入框/按钮过渡收窄为 `background-color`/`border-color`；移除按钮 hover 缩放与亮度滤镜。

### `lite/misc.css`
- 移除卡片网格 hover 时的 `backdrop-filter: blur(2px/5px)`（海报墙扫过鼠标时的最大重绘来源之一）。
- 移除移动端 OSD 的 `font-size` 过渡（字体尺寸过渡会触发布局）。

### `lite/slideshow.css`
- 交叉淡化 1.2s → 0.4s；移除内容 `blur(4px)` 滤镜过渡与 0.2–0.8s 级联延迟。
- 按钮/箭头 `transition: all` → `background-color`；移除按钮 hover 放大与按钮 backdrop blur。
- 保留静态渐变遮罩、静态 drop-shadow 与基本滑动切换功能。

### `lite/12_fixes.css` / `lite/progress_bar.css` / `lite/ratings.css`
- MUI 抽屉 `blur(30px)`、进度条 `blur(4px)` 移除（保留半透明背景）。
- MUI 列表项过渡收窄为 `background-color`/`color`；评分徽章 `transition: all` → `transform`（保留 hover 微放大反馈，且已有 `prefers-reduced-motion` 兜底）。

### 第二轮（Round 2）——默认功能补齐
- **详情页按钮文字（默认启用）**：`lite/theme.css` 新增 `@import url("../icontext.css")`，**原版规则直接复用、未重写**。效果：详情页 Trailer / Mark-as-watched / Shuffle / More / User-rating 等按钮由图标改为显示 `title` 文本；Play 按钮保留播放图标（原版行为）。该文件 0 动画、0 过渡，不产生渲染开销。
- **彩色家长分级（默认启用）**：`lite/theme.css` 新增 `@import url("ratings.css")`（Lite 版，规则与原版一致，仅 hover 过渡收窄）。根因排查结论：
  - 原版彩色规则**未在 Lite 文件中遗漏**——`lite/ratings.css` 内 268 条 `[rating='…']` 彩色规则完好（与原版逐行一致）；
  - 问题是 `ratings.css` 此前**不在默认加载链**（原版同样是可选组件），且着色依赖 `scripts/rating.js` 把 `rating` 属性写入徽章元素；
  - 本轮把 Lite 版加入默认链，**未新增任何 `!important` 掩盖**；不运行 `rating.js` 时显示灰色兜底（`#607D8B`，与原版行为一致）。

### 第三轮（Round 3）——Logo 方案定稿（mask-image 单色方案）
- **实现方式（不变）**：Logo 是 **PNG 图片**，通过每个配色文件定义的 `--logo` 变量加载，被三处消费（头部/页面标题 `.pageTitleWithDefaultLogo`、登录页 `#loginPage h1::before`、12+ 顶栏 MUI 按钮 `startIcon::before`）。
- **新机制**：`lite/logo-remix.css`（默认导入）只定义变量；`lite/theme.css` **末尾追加**覆盖块（元素规则必须放在文件末尾才能赢过级联）。三个 Logo 表面由两层构成：底层原版 PNG（`--logo-base-image`，Jellyfish / Streamberry 模式使用）与上层 mask 叠加层（用户自绘 SVG `--logo-mask-source` 作 `mask-image`，以主题色 `--logo-remix-color`（默认 `--primary-accent-color`）填充）。
- **显示策略（修复双重 Logo 盖印）**：remix 模式 `--logo-base-image: none`——**隐藏底层原版 PNG，纯主题色显示**，原版 PNG 不会从 SVG 镂空处透出。CSS 无法检测 mask 是否加载成功（[w3c/fxtf-drafts #130](https://github.com/w3c/fxtf-drafts/issues/130)），"隐藏 PNG"与"SVG 缺失自动回退 PNG"**不可兼得**：SVG 缺失时 Logo 区域为空，**手动回退** = 切换到 Jellyfish / Streamberry 模式（一行）。
- **与配色完全解耦**：底层 PNG 取自 `--logo-base-image`（remix 模式为 `none`），**不再读取配色文件的 `--logo`**；默认模式 = Nico Remix，可一键切换原版 Jellyfish / Streamberry（三种模式的可复制块见 `lite/logo-remix.css`）。使用 jellyflix 配色也不会强制换 Logo。
- **尺寸/位置沿用原版**：叠加层为绝对定位 + `pointer-events:none`，不改布局；`mask-size` 分别镜像原版几何（页面标题 `100%`、登录页/12+ 顶栏 `contain`，登录页叠加层精确对齐原 `h1` 内容盒）。
- 未改动任何上游文件；`lite/logos/` 存放说明文档与用户提供的 SVG（已就位）。

## 四、Logo 二次开发（Remixed by Nico）

> **实现方式报告**：Logo 是 **PNG 图片**（非 CSS/SVG 绘制）。上游文件 `logos/jellyfish.png`（640×271，RGBA），通过每个 `colors/*.css` 里定义的 `--logo` 变量加载（14/15 个配色指向 `jellyfish.png`，`jellyflix.css` 指向 `streamberry.png`）。`--logo` 被三处消费：① 头部/页面标题默认 Logo（`.pageTitleWithDefaultLogo`）；② 登录页 Logo（`#loginPage h1::before`）；③ Jellyfin 12+ 顶栏服务器 Logo（MUI toolbar）。**详情页 Logo 使用条目自身美术图，不受影响。**

### 方案（已定稿）：CSS mask-image 单色叠加（remix 纯彩色显示）

- **SVG 由你自行绘制**，Lite 只提供接入机制（不覆盖上游任何文件）。
- **机制**：`lite/logo-remix.css` 定义变量（模式 + 路径），`lite/theme.css` 末尾覆盖块执行元素规则。三处 Logo 由两层构成：底层原版 PNG（`--logo-base-image`）与上层 mask 叠加层（`--logo-mask-source` + `--logo-remix-color`，默认 `--primary-accent-color`）。**remix 模式底层为 `none`**——只显示主题色 mask，原版 PNG 完全隐藏、不会透出（无双重 Logo 盖印）。尺寸/位置沿用原版（`mask-size` 镜像原几何），叠加层 `pointer-events:none`，不改布局、不挡点击。
- **接入步骤（你只需做 2 步）**：
  1. 按 `lite/logos/README.md` 的绘制要求制作 SVG（单色剪影、透明背景、viewBox 建议 2.36:1 即 640×271、**不留透明留白**），保存为 `lite/logos/jellyfish-remixed.svg`。
  2. 放入后**无需改任何 CSS**——默认模式即 Nico Remix，刷新即生效。
- **回退说明**：CSS 无法检测 mask 是否加载成功，"隐藏 PNG 防双重 Logo"与"SVG 缺失自动回退 PNG"不可兼得。remix 模式下 SVG 缺失时 Logo 区域**为空**；**手动回退** = 切到 Jellyfish / Streamberry 模式（一行，见下）。
- **与配色完全解耦**：底层 PNG 取自 `--logo-base-image`（remix 为 `none`），不读取配色文件的 `--logo`；切换配色（含 jellyflix）不会强制改变 Logo。
- **独立切换 Logo 模式**：编辑 `lite/logo-remix.css` 顶部 `:root` 的三个变量（文件内有 Nico Remix / 原版 Jellyfish / Streamberry 三种模式的可复制块），例如切到 Streamberry：
  ```css
  --logo-base-image: url("../logos/streamberry.png");
  --logo-mask-source: none;
  --logo-remix-color: transparent;
  ```
  原版 PNG 的兼容显示方式（`background-image` 直接加载）在 Jellyfish / Streamberry 模式下完整保留（PNG 未必适合直接当单色 mask 用）。
- **级联说明**：元素覆盖规则位于 `lite/theme.css` **文件末尾**（@import 的内容会先于本文件自身规则处理，规则不能放在 `logo-remix.css` 里）。若在用户 Display 设置里**另加**配色导入，只换变量、不换 Logo，无需额外处理。

## 五、配色说明与切换

- **完整性确认**：原版 `colors/` 目录 15 个配色文件**全部原样保留、未改动**（每个均为纯 `:root` CSS 变量集，无动效、无 `@import`、无高开销属性）。`git status` 显示该目录从未被修改。
- **可用配色（15）**：`aurora`、`banana`、`coal`、`coral`、`forest`、`grass`、`jellyblue`、`jellyflix`、`jellypurple`、`lavender`、`midnight`、`mint`、`ocean`、`peach`、`watermelon`。
- **当前默认配色**：`jellyblue`（由 `lite/theme.css` 中的 `@import url("../colors/jellyblue.css")` 决定）。
- **切换方式**：
  - **全局（服务器）**：编辑 `lite/theme.css` 中的配色 `@import` 行，换成目标配色，例如：
    ```css
    @import url("../colors/ocean.css");   /* 全局默认改为 Ocean */
    ```
  - **单用户（Display 设置）**：在用户的 Custom CSS Code 里，先导入 `lite/theme.css`，**再**追加一行目标配色覆盖即可（后导入的 `:root` 变量生效）：
    ```css
    @import url("https://cdn.jsdelivr.net/gh/<你的仓库>/Jellyfish/lite/theme.css");
    @import url("https://cdn.jsdelivr.net/gh/<你的仓库>/Jellyfish/colors/peach.css"); /* 该设备用 Peach */
    ```
  - **动态切换（脚本）**：上游 `scripts/theme_selector.js` 在用户设置里提供下拉切换（含随机每日主题）。注意该脚本内置的是**上游 CDN 链接**，只加载"纯变量"的配色文件，**不会重新加载任何主题样式**——因此用它切换配色不会引入未经优化的原版 `theme.css`；若自托管分叉仓库，建议把脚本里的 URL 换成你自己的（`scripts/` 为上游文件，本轮未改动）。
- **性能保证**：配色文件只定义 CSS 变量，切换配色**永远不会重新加载 `lite/theme.css`**，已验收的性能优化不受影响。

## 六、静态验证结果（本机执行）

| 指标 | 原版 | Lite | 说明 |
|---|---|---|---|
| `backdrop-filter` 模糊声明 | 31 | **0** | 仅剩 4 处 `none/unset` 抵消规则 + 1 处 `@supports` 探测 |
| `@keyframes` / `animation:` 声明 | 6 / 8 | **0 / 0** | 全部移除（含常驻背景滚动动画） |
| `transition: all` | 4 | **0** | 全部收窄为具体属性 |
| `will-change` | 2 | **0** | 全部移除 |
| `filter: blur(` | 33 | **0** | 仅登录页保留 `brightness(0.5)` 压暗 |
| 花括号配对 | — | 全部 lite 文件通过 | 结构完整 |
| `@import` 引用存在性 | — | 全部通过 | 相对路径目标均存在（含 icontext/ratings/logo-remix，全部默认启用） |
| 原版文件 | — | 未改动 | `git status` 仅显示新增 `lite/` 与 `AUDIT.md` |

> 以上静态指标已由真实 Jellyfin 环境的人工浏览器验收确认与运行时表现一致（详见「七、人工验收结果」）。

## 七、人工验收结果（已完成）

**验收结论**：已在真实 Jellyfin 环境中完成 A/B 对比验收（A = 原版 `theme.css`，B = `lite/theme.css`，同一浏览器、同一账号、同一库页面切换导入并刷新）：

- ✅ **性能提升确认**：海报墙快速滚动、剧集列表、详情页、登录页、移动端布局的滚动均更流畅；DevTools Performance 中长任务与渲染耗时下降；`backdrop-filter` 不再出现在 Layers/Compositing 面板。
- ✅ **视觉等效确认**：除**动效与模糊等元素按预期减少**外，其余外观、布局、配色、字体与交互行为均与原版等效（"磨砂感"→"半透明"、"入场动画"→"加载即就位"属预期差异）。
- ✅ **功能与交互正常**：播放、预告片、切换已观看状态、详情页按钮文字、彩色家长分级、Nico Remix Logo 等均正常。

**方法（已执行）**：A = 只导入原版 `theme.css`（+你常用的可选组件）；B = 只导入 `lite/theme.css`（+对应 lite 可选组件）。同一浏览器、同一账号、同一库页面切换导入并刷新，在 DevTools Performance 录制下对比。

**各轮复验项结果**：
- **第二轮**：详情页按钮文字 ✅；彩色家长分级（配合 `scripts/rating.js`）✅；Logo 回退态 ✅；配色切换 ✅（Network 无额外 `theme.css` 请求）。
- **第三轮**：Nico Remix Logo 生效 ✅（纯主题色、无双重 Logo 盖印、尺寸位置一致）；Logo 模式切换与配色解耦 ✅；功能与布局 ✅（顶栏按钮可点击、移动端正常）；回退健全性 ✅。
- **已评估并保留**：条目详情内容区的深色半透明底块（`--secondary-background-transparent`，20% 主色）为原版既有设计（叠加 Jellyfin 基础样式），Lite 与原版一致；验收后决定**保留原样、不做改动**。

## 八、许可与署名

- 上游：https://github.com/n00bcodr/Jellyfish ，Copyright (C) 2025 n00bcodr，许可协议为仓库根目录 `LICENSE.md`（DBAD Public License）。
- 本 Lite 变体**保留原作者署名与许可证**，每个文件头部均有变体说明；修改未改变任何上游文件的授权条款。
- 若需分发，请遵循 DBAD 许可（保留署名、不得以原样改名售卖等）。
