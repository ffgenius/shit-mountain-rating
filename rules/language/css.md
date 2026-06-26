# CSS 语言规范

CSS / SCSS / Less 样式专项检查。

## 检测项

- `!important` 滥用（用 !important 覆盖 !important，比赛谁更狠）
- 选择器嵌套过深（>4 层，`#app .main .sidebar .nav ul li a`）
- ID 选择器用于样式（`#header` 权重太高，无法覆盖）
- 内联样式（`<div style="...">` 无法复用、不可维护、权重最高）
- Magic Number（`top: 37.5px;` 这个 37.5 是从哪算出来的？）
- z-index 竞赛（`z-index: 9999`、`z-index: 99999`、`z-index: 2147483647`）
- 重复的 CSS 声明块（多个文件里写着几乎一样的颜色/间距/字体）
- 未使用的 CSS（整个项目 30% 的样式从未被引用）
- 选择器特异性战争（用 `#a #b #c` 对抗 `.class`，而 `.class` 用 `!important` 反击）
- 颜色值不统一（同一个颜色用 `#333`, `#333333`, `rgb(51,51,51)`, `$dark-gray` 各写一遍）
- 硬编码断点（`@media (max-width: 1367px)` 那么这个 1367 是哪来的？）
- 不合理的 `*` 通配符重置（`* { margin: 0; padding: 0; }` 抹掉所有浏览器默认再手动加回来）
- `px` 值无处不在（该用 em/rem/百分比的地方全写 px，无障碍和响应式全挂）
- 字体未提供 fallback（`font-family: 'PingFang SC'` 没了，页面变成 serif）
- `@import` 在 CSS 中（阻塞渲染，应用 `<link>` 或构建工具导入）
- 不合理的 `float` 布局（不用 Flexbox/Grid 还在用 float，然后 clearfix 写了三遍）
- `margin-top: 999px` 定位元素（用 margin 把元素推到某个位置而不是用定位或布局）
- 不设 `box-sizing: border-box`（padding 把元素撑破，然后再算一遍 width）
- 动画无 `prefers-reduced-motion` 适配（对动效敏感的用户感觉在坐过山车）
- `overflow: hidden` 裁剪重要内容（为了清除浮动把内容也切了）
- `outline: none` 不给替代（键盘用户不知道当前焦点在哪）
- `background` 简写覆盖了之前设置的 `background-size` 等属性
- 用 `display: none` 隐藏可聚焦元素（Tab 键会聚焦到不可见的元素）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| `!important` 滥用 | 严重 | 每加一个 !important 就废掉一个修改入口 |
| 内联样式 | 严重 | 不可复用、不可覆盖、权重最高 |
| z-index 竞赛 | 严重 | z-index 超过 100 说明层级管理彻底失控 |
| 未使用的 CSS | 严重 | 白送的流量和渲染时间 |
| 选择器嵌套过深 | 高 | 超过 4 层的选择器既是性能问题也是可维护性灾难 |
| ID 选择器 | 高 | 权重太高，任何地方想覆盖都得写更长的选择器链 |
| 重复声明块 | 高 | 同一个颜色或间距在多处定义，改一处漏十处 |
| 颜色值不统一 | 高 | 同一个颜色在项目里有五种写法 |
| 无 box-sizing | 高 | 计算 width 成为数学题 |
| 字体无 fallback | 高 | 字体加载失败时页面文字面目全非 |
| 特异性战争 | 高 | 选择器和 !important 的军备竞赛 |
| outline: none 无替代 | 中 | 违反无障碍标准 |
| 动画无 prefers-reduced-motion | 中 | 无障碍问题 |
| px 不用 em/rem | 中 | 响应式和无障碍困难 |
| float 代替 Flex/Grid | 中 | 布局脆弱，clearfix 满地 |
| @import 在 CSS | 中 | 阻塞渲染，拖慢首屏 |
| 硬编码断点 | 中 | 响应式断点一个设备一个数 |
| * 通配符重置 | 中 | 过度重置然后手动恢复 |
| display:none 隐藏可聚焦元素 | 中 | Tab 到隐身元素 |
| Magin Number | 中 | 来源不明的数值 |
| margin 推元素 | 低 | 能用但脆弱 |
| overflow:hidden 切内容 | 低 | 通常在特定场景出现 |
| background 简写覆盖 | 低 | CSS 特性，理解即避免 |
