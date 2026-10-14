# lite/logos — Nico Remix Logo 放置目录

> ⚠️ **放置位置（易错）**：文件必须放在**本目录** `lite/logos/jellyfish-remixed.svg`，**不是**仓库根目录的 `logos/`（那是存放原版 `jellyfish.png` / `streamberry.png` 的上游目录）。原因：`lite/logo-remix.css` 中的 mask 路径 `url("logos/jellyfish-remixed.svg")` 是**相对 `lite/` 目录解析**的（相对 URL 按"定义该变量的样式表"的基准解析）→ 浏览器实际请求 `lite/logos/jellyfish-remixed.svg`。放错目录会导致 mask 404 → 回退显示原版 PNG（看起来"未生效"）。

本目录承载**由你（用户）自行绘制**的 Nico Remix Logo 资源。Lite 只提供接入机制，不代绘、不改上游文件。

## 资源文件

| 文件 | 用途 |
|---|---|
| `jellyfish-remixed.svg` | **你提供的 SVG**（已就位）。无需改任何 CSS —— Lite 的默认 Logo 模式（Nico Remix）即自动生效。 |
| `jellyfish.png` / `streamberry.png` | 上游原版 PNG（位于仓库根目录 `logos/`，只读）。作为回退显示，以及「原版 Jellyfish / Streamberry」两种 Logo 模式的来源。 |

## SVG 绘制要求（重要）

实现方式：CSS `mask-image` 加载该 SVG，用 SVG 的**透明度通道（alpha）**决定形状，再以主题颜色填充（默认 `--primary-accent-color`，可在 `lite/logo-remix.css` 的 `--logo-remix-color` 覆盖）。因此：

1. **单色剪影即可**：画一个实心填充的剪影、透明背景。SVG 内部用什么颜色无所谓（mask 只取 alpha 通道，显示颜色由主题变量决定）；**不要依赖** SVG 自身的多色/渐变。
2. **viewBox 宽高比建议 2.36 : 1**（与原版 `logos/jellyfish.png` 640×271 一致）。`mask-size: contain` 按此比例缩放；与之一致时显示尺寸与原版完全相同。
3. **不要留透明留白/内边距**：图形应尽量撑满 viewBox 边界。若四周留白，彩色 Logo 会比原版小，且可能露出底层的回退 PNG（出现"双重 Logo"）。
4. 建议图形外轮廓**不小于**原版 PNG 的可见区域，确保完全盖住底层回退 PNG。
5. 导出为标准独立 SVG（`<svg xmlns="http://www.w3.org/2000/svg" viewBox="…">`），无外部引用（不要依赖外部字体/图片）。

## 生效与回退行为

- **SVG 已放入（当前状态）**：三处 Logo 显示**纯主题色 Nico Remix**（mask 取 SVG 的 alpha 通道，描边与填充均显示；你的文件是描边线稿 → **主题色线稿风**）。**原版 PNG 被完全隐藏、不会透出**（无"双重 Logo"盖印）；详情页条目 Logo 不受影响。
- **回退（SVG 缺失/404 时）**：remix 模式隐藏了底层 PNG（CSS 无法检测 mask 是否加载成功，"隐藏 PNG 防双重 Logo"与"缺失时自动显示 PNG"不可兼得），因此 SVG 缺失时三处 Logo 区域**为空**。**手动回退**：把 `lite/logo-remix.css` 切到 Jellyfish / Streamberry 模式（一行），或临时把 `--logo-base-image` 设回 `url("../logos/jellyfish.png")`。
- **切换 Logo 模式**（Nico Remix ⇄ 原版 Jellyfish ⇄ Streamberry）：编辑 `lite/logo-remix.css` 顶部的 `:root` 三个变量即可（文件内有三种模式的可复制块）。Logo 与配色**完全解耦** —— 切换配色（包括 jellyflix）不会强制改变 Logo。

## 验收提示

- 放入 SVG 后刷新，检查三处 Logo 显示主题色剪影、尺寸位置与原版一致、无双重 Logo。
- 若个别浏览器出现"彩色实心方块"（对失败 mask 的行为不同），在 `lite/logo-remix.css` 设 `--logo-remix-color: transparent` 即回到纯 PNG 显示。
