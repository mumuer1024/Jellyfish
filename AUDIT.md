# Jellyfish 主题性能审计报告（轻量化前置审计）

- **审计对象**：本仓库（克隆自 https://github.com/n00bcodr/Jellyfish ，branch `main`，提交 `a9d3833`，工作树在审计时干净）
- **审计日期**：2026-10-14（本地）
- **审计方式**：只读静态分析（grep/计数/上下文阅读/diff）。**未修改任何原版文件**。
- **目标**：为"保留功能、削减动效、降低渲染压力"的 Lite 变体提供依据；本报告只列**性能嫌疑项**（高开销候选），不直接认定卡顿根因——是否构成实际卡顿需经浏览器 A/B 验证（见第五节）。

---

## 一、范围

| 类别 | 文件 | 行数 | 说明 |
|---|---|---|---|
| 主样式 | `theme.css` | 2968 | 唯一核心样式表；含 12_fixes 合并段与 dashboard 段；顶部 5 个 `@import` |
| 可选组件 | `loginpage.css`、`misc.css`、`slideshow.css`、`12_fixes.css`、`progress_bar.css`、`ratings.css`、`indicators.css`、`icontext.css`、`sidebar.css`、`scrollbar.css`、`card_buttons.css`、`streamberry_logo.css`、`bg_scroll.css` | — | 按需导入 |
| 大文件 | `arr-tags-links.css` | 6620 | 纯外部链接图标映射（TMDB 图 URL），**无动效**，默认不导入 |
| 资源 | `colors/`（16 套，纯 `:root` 变量）、`fonts/`（woff2，`font-display: swap`）、`logos/` | — | 无动效 |
| 脚本 | `scripts/`（6 个 JS） | — | 功能脚本，非本次范围 |

## 二、方法

- 统计 `animation`/`@keyframes`/`transition`/`backdrop-filter`/`filter`/`will-change`/`transform`/大 `box-shadow` 的分布与上下文；
- 对每个高开销声明逐一读取所在规则，判断其作用面（面积、是否 hover、是否常驻、动画目标属性）；
- 与优化计划（见 `lite/README.md`）逐条对照。

## 三、性能嫌疑项清单（按可疑度排序，均待 A/B 验证）

### 1. backdrop-filter 玻璃拟态（35 处，高嫌疑）
- `theme.css` 28 处、`misc.css` 2 处、`progress_bar.css` 2 处、`12_fixes.css`/`loginpage.css`/`slideshow.css` 各 1 处。
- 其中模糊半径：`blur(50px)`×6、`blur(30px)`×3、`blur(25px)`×2、`blur(100px)`×1、`blur(10px)`×2、`blur(var(--blur))`（默认 15px）×多处；另 4 处为 `none/unset` 抵消规则。
- 作用于大面积元素：headerTabs/headerRight 药丸（`theme.css:488`）、桌面/移动抽屉（`640/664`）、移动端头部（`777/794`）、背景容器（`308`）、对话框（`2886`）、MUI 抽屉（`2635`）。
- **为何可疑**：`backdrop-filter` 强制把元素背后的整块区域独立合成并逐帧重新模糊；滚动或背景变化时，该区域越大、半径越大，逐帧成本越高——低端 GPU 上是最常见的卡顿来源之一。**嫌疑点**：滚动时头部/抽屉/背景的模糊重算、hover 时卡片网格上的模糊（`misc.css`）。

### 2. 常驻无限动画（高嫌疑）
- `bg_scroll.css:3`：`.backdropImage { animation: backgroundScroll 50s linear infinite; }`，逐帧动画 `background-position`（0%→100% 往复）。
- **为何可疑**：全屏背景图每帧重绘/合成，且永不停止，是恒定开销而非瞬时开销。

### 3. 详情页入场/装饰动画（中嫌疑）
- `theme.css` 1500 段：`.detailLogo` 滑入 0.8s、`.detailPagePrimaryContainer` 淡入滑入 **1.5s**、`.starRatingContainer:hover .starIcon` 旋转 1s、评分 hover 倾斜、flat/link 按钮 hover 放大。
- 移动端背景图淡入 0.8s（`theme.css:2211`）。
- **为何可疑**：加载与 hover 时刻的合成/动画时长偏长；1.5s 的入场动画在低端设备上可感知。

### 4. 卡片网格 hover 触发 backdrop blur（中嫌疑）
- `misc.css:157/160`：`.cardOverlayContainer:has(.cardOverlayButton:hover)` blur(5px)、`.cardOverlayContainer:hover` blur(2px)；`theme.css:2190` 移动端导航 hover blur。
- **为何可疑**：海报墙网格中鼠标扫过会反复触发"模糊区域建立→撤销"，是滚动/hover 卡顿的重点嫌疑。

### 5. 登录页（中嫌疑）
- 全屏背景 `filter: blur(5px) brightness(0.5)`（`loginpage.css:22`）+ `fadeIn 1s` + `will-change: opacity`（`29`）；用户卡片 `will-change: transform, box-shadow`（`84`）+ hover 50px 辉光（`96`）。
- **为何可疑**：全屏模糊为一次性大开销；`will-change` 为每张卡片常驻预留图层（内存/合成压力）。

### 6. slideshow.css（中嫌疑，仅当使用幻灯片首页）
- `transition: all`（按钮/箭头）、内容 `filter: blur(4px)` 过渡 + 0.8s、交叉淡化 1.2s + 0.2–0.8s 级联延迟、按钮 backdrop blur。
- **为何可疑**：`transition: all` 使任意属性变化都动画；blur 滤镜过渡逐帧重绘；级联延迟拉长合成期。

### 7. 过渡目标为非 GPU 属性（低-中嫌疑）
- `transition: all`/裸缩写（`.2s`/`0.2s`）共 18 处；`font-size` 过渡（`misc.css` OSD）；`box-shadow` 过渡（`theme.css:999/1043/1799`）；`filter` 过渡（`theme.css:2126`）；`text-shadow` 过渡（`2134`）。
- **为何可疑**：font-size/box-shadow/filter/text-shadow 的过渡逐帧触发重绘/布局，而非仅合成；裸缩写会覆盖"任何"属性变化。

### 8. 装饰性 hover 放大与发光（低嫌疑）
- `scale(1.05~1.1)` 多处、`0 0 30–50px` 辉光（skip 按钮、登录卡片）。

## 四、无嫌疑、保持原样的部分
- 全部布局/配色/圆角/字体/隐藏规则/响应式规则；`colors/` 纯变量；`fonts/` 使用 woff2 + `font-display: swap`；无动效的可选组件；`arr-tags-links.css`（无动效，默认不导入）；`scripts/`。
- 已保留的必要反馈：抽屉/菜单滑入（transform/left，瞬时）、头部固定过渡、进度条标记 opacity 反馈、Up Next 弹窗按钮 transform 反馈、alpha 选择器反馈、`ratings.css` 的 `prefers-reduced-motion` 兜底。

## 五、验证建议（A/B，人工浏览器）

1. **A/B 方法**：同一 Jellyfin 实例，A=原版导入，B=`lite/theme.css` 导入；同账号、同页面、同窗口尺寸；DevTools Performance 录制对比。
2. **重点场景**：① 海报墙滚动与鼠标扫过卡片网格；② 剧集列表；③ 详情页首屏与评分 hover；④ 登录页（全屏模糊 vs 压暗）；⑤ 移动端（窄窗口）头部药丸、抽屉、OSD。
3. **观察指标**：滚动/悬停 FPS、长任务与渲染耗时、Layers 面板中 `backdrop-filter` 图层是否消失、主观视觉差异清单（磨砂→半透明属预期差异）。

## 六、结论
- 上述 1–8 为**高开销嫌疑项**，全部已由 Lite 变体消除或收窄（见 `lite/README.md` 变更清单与静态验证表）。
- 静态层面：Lite 与原件仅在动效/滤镜/发光/过渡上不同，功能规则零改动（逐行 diff 核对）。
- **未验证项**：真实浏览器渲染、帧率实测、各浏览器/设备差异——需按第五节人工验证。

---

## 七、补充记录：Round 2 / Round 3 / Round 4（2026-10-14 迁入）

> 以下内容原位于 `lite/README.md`「三、变更清单」「六、静态验证结果」「七、人工验收结果」，整理公开发布文档时移入本内部文件；公开文档仅保留摘要与链接（见 `lite/README.md` 与 `lite/README.zh-CN.md`）。

### 7.0 变更清单（Round 1 —— 性能轻量化明细）

按"等价视觉轻量化"原则，仅移除/收窄动效与渲染开销，**不涉及任何布局、配色、字体、交互行为、插件适配或响应式改动**。

**全局**
- **移除全部 `backdrop-filter` 玻璃拟态模糊**（原版 35 处，含 `blur(50px)`×6、`blur(30px)`×3、`blur(100px)`、`blur(15px)` 默认等多处大面积元素）：改为原有半透明纯色背景（如 `var(--secondary-background-transparent)`）。保留 4 处 `backdrop-filter: none/unset`（用于抵消 Jellyfin 默认样式）与 1 处 `@supports` 特性探测，不影响功能。
- **移除常驻动画**：`bg_scroll.css` 的 `backgroundScroll 50s linear infinite`（全屏背景逐帧重绘）→ 静态展示。
- **移除全部 `will-change`**（2 处，登录页）：不再为静态/悬停元素预留常驻图层。
- **移除大辉光阴影（≥20px）**：登录卡片 50px 辉光、skip 按钮 30px 辉光等（保留常规小投影与功能性阴影）。
- **`transition: all` 与裸缩写逐处清理**：按"菜单、弹窗、头部固定等必要状态反馈保留，纯装饰过渡移除"处理（明细见各文件）。

**`lite/theme.css`**
- 移除 24 处 backdrop-filter 模糊声明。
- 移除详情页动效：logo 滑入（0.8s）、主容器淡入滑入（1.5s）、星级 hover 旋转（1s）、评分 hover 倾斜、flat/link 按钮 hover 放大。
- 移除移动端背景图淡入动画（0.8s）。
- 过渡收窄：侧边栏列表项、Up Next 按钮、MUI 侧栏/抽屉项、fab/raised、引导频道图片（filter 过渡删除）、日期标签页等改为具体廉价属性（color/background/opacity/transform）。
- 保留：菜单/抽屉滑入（transform/left）、头部固定过渡（`.5s`）、进度条标记 opacity 反馈、alpha 选择器反馈等。
- 导入路径改为相对路径。

**`lite/loginpage.css`**
- 全屏背景 `filter: blur(5px) brightness(0.5)` → `filter: brightness(0.5)`（去全屏模糊，保留压暗以保证文字可读）。
- 移除登录淡入动画与 `will-change`。
- 移除用户卡片 hover 缩放+辉光（保留 `:focus-within` 轮廓，无障碍不受影响）。
- 对话框/输入框/按钮过渡收窄为 `background-color`/`border-color`；移除按钮 hover 缩放与亮度滤镜。

**`lite/misc.css`**
- 移除卡片网格 hover 时的 `backdrop-filter: blur(2px/5px)`（海报墙扫过鼠标时的最大重绘来源之一）。
- 移除移动端 OSD 的 `font-size` 过渡（字体尺寸过渡会触发布局）。

**`lite/slideshow.css`**
- 交叉淡化 1.2s → 0.4s；移除内容 `blur(4px)` 滤镜过渡与 0.2–0.8s 级联延迟。
- 按钮/箭头 `transition: all` → `background-color`；移除按钮 hover 放大与按钮 backdrop blur。
- 保留静态渐变遮罩、静态 drop-shadow 与基本滑动切换功能。

**`lite/12_fixes.css` / `lite/progress_bar.css` / `lite/ratings.css`**
- MUI 抽屉 `blur(30px)`、进度条 `blur(4px)` 移除（保留半透明背景）。
- MUI 列表项过渡收窄为 `background-color`/`color`；评分徽章 `transition: all` → `transform`（保留 hover 微放大反馈，且已有 `prefers-reduced-motion` 兜底）。

### 7.1 变更清单（Round 2 —— 默认功能补齐）

- **详情页按钮文字（默认启用）**：`lite/theme.css` 新增 `@import url("../icontext.css")`，**原版规则直接复用、未重写**。效果：详情页 Trailer / Mark-as-watched / Shuffle / More / User-rating 等按钮由图标改为显示 `title` 文本；Play 按钮保留播放图标（原版行为）。该文件 0 动画、0 过渡，不产生渲染开销。
- **彩色家长分级（默认启用）**：`lite/theme.css` 新增 `@import url("ratings.css")`（Lite 版，规则与原版一致，仅 hover 过渡收窄）。根因排查结论：
  - 原版彩色规则**未在 Lite 文件中遗漏**——`lite/ratings.css` 内 268 条 `[rating='…']` 彩色规则完好（与原版逐行一致）；
  - 问题是 `ratings.css` 此前**不在默认加载链**（原版同样是可选组件），且着色依赖 `scripts/rating.js` 把 `rating` 属性写入徽章元素；
  - 本轮把 Lite 版加入默认链，**未新增任何 `!important` 掩盖**；不运行 `rating.js` 时显示灰色兜底（`#607D8B`，与原版行为一致）。

### 7.2 变更清单（Round 3 —— Logo 方案定稿，mask-image 单色方案）

- **实现方式（不变）**：Logo 是 **PNG 图片**，通过每个配色文件定义的 `--logo` 变量加载，被三处消费（头部/页面标题 `.pageTitleWithDefaultLogo`、登录页 `#loginPage h1::before`、12+ 顶栏 MUI 按钮 `startIcon::before`）。
- **新机制**：`lite/logo-remix.css`（默认导入）只定义变量；`lite/theme.css` **末尾追加**覆盖块（元素规则必须放在文件末尾才能赢过级联）。三个 Logo 表面由两层构成：底层原版 PNG（`--logo-base-image`，Jellyfish / Streamberry 模式使用）与上层 mask 叠加层（用户自绘 SVG `--logo-mask-source` 作 `mask-image`，以主题色 `--logo-remix-color`（默认 `--primary-accent-color`）填充）。
- **显示策略（修复双重 Logo 盖印）**：remix 模式 `--logo-base-image: none`——**隐藏底层原版 PNG，纯主题色显示**，原版 PNG 不会从 SVG 镂空处透出。CSS 无法检测 mask 是否加载成功（[w3c/fxtf-drafts #130](https://github.com/w3c/fxtf-drafts/issues/130)），"隐藏 PNG"与"SVG 缺失自动回退 PNG"**不可兼得**：SVG 缺失时 Logo 区域为空，**手动回退** = 切换到 Jellyfish / Streamberry 模式（一行）。
- **与配色完全解耦**：底层 PNG 取自 `--logo-base-image`（remix 模式为 `none`），**不再读取配色文件的 `--logo`**；默认模式 = Nico Remix，可一键切换原版 Jellyfish / Streamberry（三种模式的可复制块见 `lite/logo-remix.css`）。使用 jellyflix 配色也不会强制换 Logo。
- **尺寸/位置沿用原版**：叠加层为绝对定位 + `pointer-events:none`，不改布局；`mask-size` 分别镜像原版几何（页面标题 `100%`、登录页/12+ 顶栏 `contain`，登录页叠加层精确对齐原 `h1` 内容盒）。
- 未改动任何上游文件；`lite/logos/` 存放说明文档与用户提供的 SVG（已就位）。

### 7.3 变更清单（Round 4 —— 详情页深色底块评估，决定保留）

- 主题（原版与 Lite 一致）会在条目详情内容区叠加 `--secondary-background-transparent`（= 20% 主色）半透明深色块，并配合 `html { background-color: #000000 }` 页面基底（jellyblue 下 20% 亮蓝叠纯黑 ≈ 近黑观感）。
- 曾提供对比用移除开关 `lite/no_detail_bg.css`（opt-in，需在 `lite/theme.css` 之后导入），验收后用户**决定保留原块、不做改动**；该文件已回滚删除，无残留引用。
- 结论：深色块为原版既有设计（叠加 Jellyfin 基础样式），Lite 与原版一致，**保留原样**。

### 7.4 静态验证结果（本机执行）

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

### 7.5 人工验收记录（真实 Jellyfin 环境，已完成）

- **验收结论**：A/B 对比（A = 原版 `theme.css`，B = `lite/theme.css`，同一浏览器、同一账号、同一库页面切换导入并刷新）：
  - **性能**：海报墙快速滚动、剧集列表、详情页、登录页、移动端布局的滚动**主观更流畅**；DevTools Performance 中长任务与渲染耗时下降；`backdrop-filter` 不再出现在 Layers/Compositing 面板。
  - **视觉等效**：除动效与模糊按预期减少外，其余外观、布局、配色、字体与交互行为均与原版等效（"磨砂感"→"半透明"、"入场动画"→"加载即就位"属预期差异）。
  - **功能与交互正常**：播放、预告片、切换已观看状态、详情页按钮文字、彩色家长分级、Nico Remix Logo 等均正常。
- **各轮复验项**：第二轮（按钮文字 / 彩色分级配合 `scripts/rating.js` / Logo 回退态 / 配色切换无额外 `theme.css` 请求）✅；第三轮（Nico Remix Logo 生效、纯主题色无双重盖印、模式切换与配色解耦、功能与布局、回退健全性）✅。
- **注明**：性能描述为人工主观与 DevTools 观察结论（非仪器化基准测试），与公开文档中的表述口径一致。
