# 版面层：基线网格（Müller-Brockmann 法）

来源：Josef Müller-Brockmann《平面设计中的网格系统》"网格的结构"一章。核心原则：**版面不是先画框再塞字，而是从正文行距出发，算出单元格、空行和版心高度，所有图片和文字块的边缘都落在文字行上。**

顺序：行距 L → 字阶倍率 → 每格行数 r → 格高、格间距 → 版心高度 → 页边距。按序走完，每步给出数值。

---

## 第 1 步：定基数 L（正文行距）

输入：正文字号 `F`、正文行距 `L`。

- Web 默认：`F = 16px`，`L = 24px`。印刷默认 10/12pt。中文长文见第 8 节。
- `L` 是整个系统的基数，后面所有纵向尺寸都是 `L` 的整数倍（小字允许 `L/2`）。
- 只给了字号时，按 1.4–1.6 倍取一个 **4px 倍数、最好 8px 倍数**的行距：16 → 24，14 → 20 或 24，18 → 28 或 32。
- 栏宽检查：正文每行 7–10 个英文单词 / 45–75 字符 / 中文 25–40 字。超出就分栏或限宽（`max-width: 65ch`，中文用 `40em` 量级）。

---

## 第 2 步：字阶与行距倍率

### 2a. 生成字阶

输入：正文 `F`、比率 `ratio`（默认 1.25；紧凑 UI 用 1.2，展示型用 1.333 或 1.5）、向上几级、向下几级。

```
size_n = F × ratio^n   → 四舍五入到偶数 px
```

默认 F=16、ratio=1.25：12 · 14 · 16 · 20 · 24 · 32 · 40 · 48 · 64（中间按需删，一个页面用 4–5 级足够）。

### 2b. 给每级定行距

规则：**行距不是自由值，只能是 L 的倍数或 L/2。**

```
size ≤ F    →  lh = L/2 的倍数，取 ≥ size × 1.3 的最小值
size > F    →  lh = L 的倍数，取 ≥ size × 1.15 的最小值
```

L = 24 时的结果：

| 用途 | 字号 | 行距 | = 几行正文 |
|---|---|---|---|
| 辅助 / 图注 | 12 | 24（两行 12/12 也等于 24） | 1 |
| 次要正文 | 14 | 24 | 1 |
| 正文 | 16 | 24 | 1 |
| 小标题 | 20 | 24 | 1 |
| 标题 | 24 | 48（紧凑可用 24） | 2 |
| 标题 | 32 | 48 | 2 |
| 大标题 | 40 | 48 | 2 |
| 展示 | 48 | 72 | 3 |
| 展示 | 64 | 72 | 3 |

L = 32（中文长文 16/32）时：12/16 · 14/32 · 16/32 · 20/32 · 24/32 · 32/64 · 48/64。

例外：标题行距取 `L × 1.5`（如 24/36）只在**该标题固定两行或用 margin 补齐**到整行时允许——块的总高必须是 `n × L`。

书中原例（印刷）：图注 6/6pt ×2 行 = 正文 10/12pt ×1 行 = 12pt；标题 20/24pt ×1 行 = 正文 ×2 行 = 24pt。

---

## 第 3 步：每格行数与版心高度

输入：可用总行数 `N`（可用高度 ÷ L，向下取整）、纵向格数 `k`、格间空行数 `g`（默认 1；放图注时 2–3）。

```
空行总数      = (k − 1) × g
可分配行数    = N − (k − 1) × g
每格行数 r    = floor(可分配行数 / k)        ← 向下取整，排版没有半行
校正后总行数  = k × r + (k − 1) × g
版心高度      = 校正后总行数 × L
删掉的行数    = N − 校正后总行数            ← 归入页边距 / 底部留白
```

书中原例：N=57，k=4，g=1 → 57−3=54 → 54÷4=13.5 → r=13 → 4×13+3 = **55 行**，删 2 行。

Web 例：内容区 1080px，L=24 → N=45；k=4，g=1 → 45−3=42 → r=10 → 4×10+3=43 行 → 版心 1032px，剩 48px 给上下留白。

需要同一版心支持多种划分时，选能被多个 k 整除的 N。书中例子：59 行可分 12×4、10×5、6×9、5×11、4×14（格间各空 1 行）。反过来：先选 N，使 `N + g` 能被几种 `(r + g)` 整除。

---

## 第 4 步：格、栏、图片尺寸

```
格高 field_h     = r × L                        ← 图片、卡片高度
格间距 gap_y     = g × L                        ← row-gap，也是图注位置
跨 m 格高度      = m × r × L + (m − 1) × g × L
栏宽 col_w       = (版心宽 − (c − 1) × gap_x) ÷ c  → 取整 px
```

- 栏间距 `gap_x` 可不等于 L，但建议取 `L/2`、`L`、`2L` 之一。
- 横向细分：2 栏 → 4，3 栏 → 6，4 栏 → 8 → 16。大量图文 + 图表时 4 栏最通用（书中 20 格 = 4×5，32 格 = 4×8）。12 列栅格流行就因为能被 2、3、4、6 整除。
- 图片进格：**裁切图片适应格，不是拉伸格适应图片**（`object-fit: cover`）。这是书里"摄影师按网格取景"的 Web 版。
- 格数越多越灵活，也越容易失去层次。能用 8 格解决就别用 32 格。
- 精确对齐（印刷级，可选）：图片顶边对齐大写字母顶端、底边对齐降部线，格高 = (r−1)×L + 字面高。Web 上高度取 `r × L` 即可，必要时用 `text-box-trim` / 负 margin 修正首行半行距。

---

## 第 5 步：页边距

书中原则：四边相等 = 乏味欠考虑；版心太高像要溢出，太低像要掉下去；地脚 > 天头让版心上推、显得轻盈。

Web 换算（纵向仍是 L 的倍数）：

| 位置 | 规则 | L=24 示例 |
|---|---|---|
| 页面顶部 padding | `a × L` | 2L = 48 |
| 页面底部 padding | `≥ 1.5 × 顶部`，取 L 倍数 | 3L = 72 |
| 左右 padding | rem 刻度，随断点变（见第 7 步） | 16 / 24 / 32 |
| 章节之间 | `2L`–`3L` | 48–72 |
| 标题与其下内容 | `L` 或 `L/2` | 24 或 12 |

对页/书籍类（电子书、长文阅读器）可用经典比例 内:上:外:下 = 2:3:4:6，再把每个值取整到 L 的倍数。

---

## 第 6 步：输出 token

```css
:root {
  --baseline: 24px;                 /* L */
  --half-line: calc(var(--baseline) / 2);
  --field-rows: 10;                 /* r */
  --field-h: calc(var(--field-rows) * var(--baseline));
  --gap-y: var(--baseline);         /* g × L */
  --gap-x: 24px;
  --cols: 4;
  --page-pt: calc(2 * var(--baseline));
  --page-pb: calc(3 * var(--baseline));
}
body        { font-size: 16px; line-height: var(--baseline); }
h1          { font-size: 40px; line-height: calc(2 * var(--baseline)); }
h2          { font-size: 32px; line-height: calc(2 * var(--baseline)); }
h3          { font-size: 20px; line-height: var(--baseline); }
small, figcaption { font-size: 12px; line-height: var(--baseline); }

/* 写法服从 figma-structure.md：用 flex + gap，不用 grid */
.row   { display: flex; gap: var(--gap-x); }
.col   { flex: 1; display: flex; flex-direction: column; gap: var(--gap-y); }
.field { height: var(--field-h); }
.field.span-2 { height: calc(2 * var(--field-h) + var(--gap-y)); }
```

调试覆盖层（看基线是否对齐）：

```css
.debug-baseline { background: repeating-linear-gradient(transparent 0 23px, rgba(255,0,0,.15) 23px 24px); }
```

---

## 第 7 步：响应式重算

原则：**L 跨断点不变**（正文字号不变，行距就不变），变的是栏数、栏宽、r、页面左右 padding。

| 断点 | 栏数 c | 左右 padding | r 怎么定 |
|---|---|---|---|
| < 768 | 1–2 | 16 | 按手机视口重算 N，k 通常减到 2–3 |
| 768–1439 | 2–4 | 24 | 重算 |
| ≥ 1440 | 4–6 | 32 | 重算 |

- 每个断点单独跑第 3 步，得到各自的 r；写成 `--field-rows` 在媒体查询里覆盖。
- 流式栏宽下图片高度不要随宽度缩放（否则必然偏离基线）：固定 `height: var(--field-h)` + `object-fit: cover`。真的需要保持比例的图（作品截图）用 `aspect-ratio`，并把它放在自己的一行，行高不再要求整行。
- 栏数变化时重新检查每行字数（第 1 步）。

---

## 第 8 步：中文排版的差异

- 中文无基线、无升降部，行盒上下就是对齐边——比西文更容易对格。
- 正文行距比西文大：**1.7–2.0 倍**。推荐 16/28（L=28，4 的倍数）或 16/32（L=32，8 的倍数，长文更舒展）；UI 密集场景 14/24。
- 栏宽按字数算：每行 25–40 字。16px 时约 400–640px，`max-width: 40em`。
- 标题行距 1.3–1.5 倍，同样取 L 的倍数：L=28 时 24/28、32/56；L=32 时 24/32、32/64。
- 中英混排：英文和数字用同一行距；字号差 1–2px 视觉才等大（英文 15 ≈ 中文 16），但不要因此改行距。
- 段落：首行缩进 2em 或段间空 `L/2`–`L`，二选一，不同时用。
- 标点：开启 `text-autospace` / 中西文间空 1/4 em（`word-spacing` 或工具插入），不影响纵向节奏。

---

## 第 9 步：Figma 变量导出

把 token 表翻成 Figma Variables，三个集合，命名与 CSS 变量一一对应，图层命名由 `figma-export-checklist.md` 的 class 规则保证一致。

| 集合 | 变量名 | 类型 | 值（L=24 示例） | 对应 CSS |
|---|---|---|---|---|
| grid | `grid/baseline` | number | 24 | `--baseline` |
| grid | `grid/half-line` | number | 12 | `--half-line` |
| grid | `grid/field-rows` | number | 10 | `--field-rows` |
| grid | `grid/field-h` | number | 240 | `--field-h` |
| grid | `grid/gap-y` | number | 24 | `--gap-y` |
| grid | `grid/gap-x` | number | 24 | `--gap-x` |
| grid | `grid/cols` | number | 4 | `--cols` |
| spacing | `spacing/0.25` … `spacing/2` | number | 4 … 32 | rem 刻度（见 spacing.md） |
| spacing | `spacing/page-pt` / `page-pb` | number | 48 / 72 | `--page-pt` / `--page-pb` |
| type | `type/body/size` / `type/body/lh` | number | 16 / 24 | `body` |
| type | `type/h2/size` / `type/h2/lh` | number | 32 / 48 | `h2` |
| type | `type/caption/size` / `type/caption/lh` | number | 12 / 24 | `figcaption` |

- 响应式：grid 集合建三个 mode（mobile / tablet / desktop），`cols`、`field-rows`、`gap-x`、`page-px` 按 mode 填。
- 项目有自己的变量前缀（如 `--brand-*`）时，按 `.claude/layout-project.md` 的前缀映射，不改结构。

---

## 检查清单（逐条回答是/否）

1. 每种文字样式的 line-height 都是 `L` 的整数倍或 `L/2`？
2. 图片、卡片、色块高度都是 `r × L` 或跨格公式的结果？
3. 段落间距、模块间距都是整行，没有 0.5 行以外的碎值？
4. 宽栏字号 ≥ 窄栏字号？
5. 页面上下 padding 不相等（下 > 上）？
6. 色块里的文字有缩进，没有顶到边？
7. 格数没有超过需要？
8. 每个断点都单独算过 r，图片用固定高 + cover？
9. 中文页面的 L 取了 1.7 倍以上？
