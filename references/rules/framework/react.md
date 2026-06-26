# React 框架规则

React 框架专项检查。

## 检测项

- `useEffect` 滥用（依赖数组问题、缺少清理函数、一个组件十几个 Effect）
- Props Drilling（属性层层传递超过 3 层）
- Context 滥用（把所有状态都扔进 Context，一个变化全家重渲染）
- 缺少 `useMemo` / `useCallback`（每次渲染都重新计算/重建函数）
- JSX 中内联函数/对象（`onClick={() => ...}` 每次渲染新建引用）
- Render Storm（父组件重渲染导致子树全部重渲染）
- 组件过大（>300 行，一个组件承担了三个组件的职责）
- State 设计不合理（多个 useState 该用 useReducer、派生状态该直接计算）
- `useState` 泛滥（十几个 useState 在同一个组件里比变量声明还长）
- 缺少 `key` 或用 `index` 作 key（列表渲染的经典错误）
- 直接操作 DOM（useRef + querySelector / addEventListener，绕过 React）
- 废弃生命周期（componentWillMount、componentWillReceiveProps 仍在使用）
- 受控/非受控组件混用（同一个 input 既有 value 又有 defaultValue）
- 在 render 中调用 setState（条件渲染中触发状态更新死循环）
- Class Component 与 Function Component 混用不统一
- `forceUpdate` 使用（本该通过 state 驱动的更新被暴力刷新）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| Render Storm | 严重 | 每次状态更新触发 50+ 个组件无意义重渲染 |
| useEffect 滥用 | 严重 | 依赖数组为空或缺失清理函数导致内存泄漏 |
| 在 render 中 setState | 严重 | 条件分支中触发状态更新导致死循环 |
| 直接操作 DOM | 高 | 绕过虚拟 DOM，React 状态与实际 DOM 不同步 |
| 废弃生命周期 | 高 | React 17+ 已移除，代码在崩溃边缘 |
| 缺少 key 或用 index | 高 | 列表重排时渲染错误、性能下降 |
| Props Drilling | 高 | 超过三层属性传递，中间组件被迫接收无关 props |
| 组件过大 | 中 | 超过 300 行，应按功能拆分 |
| State 冗余 | 中 | useState 数量过多或派生状态未直接计算 |
| 受控/非受控混用 | 中 | React 报警告，输入行为不可预测 |
| 内联函数/对象 | 中 | 每次渲染新建引用，导致子组件不必要的重渲染 |
| Context 滥用 | 中 | 高频变化的状态放进 Context，所有消费者都被拖累 |
| 缺少 Memo | 低 | 可优化的重计算，但在关键路径之外 |
| forceUpdate | 低 | 绕过了声明式更新的意图 |
