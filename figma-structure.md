# 代码层·生成期：三条硬规则

写任何会转成 Figma 设计稿的 HTML/CSS 之前读这一页。**只有三条，写的时候遵守即可，不做走查、不输出报告**——走查在导出前另做（`figma-export-checklist.md`）。

为什么只有三条：转写工具是机械翻译，`div` → Frame，class → 图层名，CSS 属性 → Figma 属性。下面三条违反之后要改整棵 DOM 树，其它问题导出前改几行就行。

---

## 1. 布局用 flexbox，不用 CSS grid

Figma 的 Grid 列宽是写死的 track，子元素不能 Hug / Fill，多选没有统一间距字段。flex 的 gap、方向、对齐都能一一映射到 Auto Layout。

唯一例外：日历、图片墙这类行和列都必须对齐的真二维网格。用的时候在元素上方写一行注释说明为什么 flex 做不到。

`baseline-grid.md` 算出来的栏和格，用嵌套 flex 表达：外层 `display:flex; gap: var(--gap-x)` 做栏，每栏 `flex-direction: column; gap: var(--gap-y)` 做格。数值不变。

## 2. 间距只用 gap 和 padding，不用 margin

Figma 图层没有 margin 属性，转写工具只能造一个只有 padding 的空壳把元素包起来——设计稿里的垃圾层几乎全来自这里。

- 需要不同间距时，拆成不同的 flex 容器分组，每组设自己的 gap。例：标题与副标题间 12、操作区间 20 → 标题+副标题包成 gap:12 的子容器，外层 gap:20。
- 不用空 div、`<br>`、透明元素、伪元素撑间距。
- 唯一例外：`margin: 0 auto` 水平居中——这是对齐方式不是间距，会映射成居中对齐。

## 3. class 用内容语义命名

转写工具优先用 class 名给图层命名。class 起对了，设计稿图层名就是对的；起错了要在 Figma 里逐层重命名。

- 全小写、连字符：`.step-bar` `.goal-section` `.advice-card` `.metric-card`
- 禁止：`.container` `.wrapper` `.box` `.inner` `.content` `.main` `.section`
- 禁止编号：`.section-1` `.main-content-2` `.div3`
- 每个语义块最外层必须有 class，不要只靠标签选择器挂样式。
- 重复出现的结构用完全相同的 class 名和内部结构，不要每处微调。

---

顺手的两个习惯（不强制，但导出前会查）：尺寸优先 `fit-content`（→ Hug）或 `flex: 1`（→ Fill），少写死 px；单个文本直接用 `h1`–`h6` / `p` / `span`，不再包 div。
