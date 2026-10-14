# Jellyfish 主题性能审计报告（轻量化前置审计）

- **审计对象**：`/opt/dsh/jellyfish-lite/`（克隆自 https://github.com/n00bcodr/Jellyfish ，branch `main`，提交 `a9d3833`，工作树在审计时干净）
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
