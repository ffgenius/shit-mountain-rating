# Vue 框架规则

Vue 框架专项检查。

## 检测项

- Watch Hell（watch 之间链式触发形成不可预测的更新链）
- `deep: true` 深度监听滥用（对大对象深度监听导致性能暴降）
- `computed` 中有副作用（在 computed 里发请求或改状态）
- `ref` / `reactive` 误用（该用 ref 的用了 reactive，或者反过来）
- `<script setup>` 逻辑过长（超过 300 行的 setup，失去组合式 API 的意义）
- Props / Emits 未定义类型（TypeScript 项目中缺失类型，失去安全网）
- `v-if` 与 `v-for` 同时使用（优先级问题，每次渲染都先遍历再判断）
- 直接修改 Props（子组件修改父组件传入的引用类型数据）
- Mixins（Option API 的遗留模式，命名冲突和来源不明）
- 模板表达式过于复杂（`{{ items.filter(...).map(...).join(...) }}` 地狱）
- `$nextTick` 滥用（用 nextTick 解决本该由响应式系统处理的问题）
- watch 无清理函数（watch 回调中注册了定时器/事件但未在 onCleanup 中清除）
- 生命周期钩子顺序依赖（onMounted 在 setup 中的书写顺序决定执行顺序）
- template ref 在 mounted 前使用（在 onBeforeMount 或 setup 顶层访问 ref.value）
- Composable 函数名不以 `use` 开头（违反 Vue 约定，降低可读性）
- `provide` / `inject` 未提供默认值（依赖注入无回退，深层组件可能拿到 undefined）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| 直接修改 Props | 严重 | 单向数据流被打破，数据变化无法追踪 |
| computed 副作用 | 严重 | 计算属性有副作用，渲染顺序不可控 |
| v-if 与 v-for 同用 | 严重 | 性能问题且逻辑意图模糊 |
| Watch Hell | 高 | watch 互相触发，一个操作引发不可预测的连锁反应 |
| 深度监听滥用 | 高 | `deep: true` 导致页面卡顿 |
| Mixins | 高 | 命名冲突和来源不明，修 BUG 像解谜 |
| 模板表达式复杂 | 中 | 逻辑写进模板，不可测试不可复用 |
| ref/reactive 误用 | 中 | 响应式 API 选择不当导致 .value 遍地或响应性丢失 |
| `<script setup>` 过长 | 中 | 超过 300 行，组合式 API 的优势被淹没 |
| Props/Emits 无类型 | 中 | 缺少编译时检查，重构时心惊胆战 |
| $nextTick 滥用 | 中 | 用异步回调掩盖同步逻辑本该解决的数据流问题 |
| 生命周期顺序依赖 | 低 | 脆弱且难以调试 |
| Composable 命名 | 低 | 可读性问题，但功能不受影响 |
