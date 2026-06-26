# JavaScript 语言规范

JavaScript 语言专项检查。

## 检测项

- `var` 声明（应使用 `let` / `const`，函数作用域导致提升和泄漏问题）
- `==` 而非 `===`（隐式类型转换，`[] == ![]` 返回 `true`）
- 全局变量污染（未加 `var/let/const` 直接赋值，或挂在 window 上）
- `eval()` 使用（动态执行字符串代码，安全和性能双重灾难）
- `with` 语句（已废弃，作用域不可预测）
- `new Function()` 构造器（等价于 eval，动态创建函数）
- 回调地狱（回调嵌套 >3 层，不用 Promise/async-await）
- 未处理的 Promise（new Promise 不 catch，async 函数调用不加 await 也不 catch）
- `for...in` 遍历数组（遍历的是键名而非值，还会遍历原型链）
- `delete` 操作数组元素（不会调整 length，留下空洞）
- `arguments` 对象使用（应使用 rest 参数 `...args`）
- 修改内置对象原型（`Array.prototype.myMethod = ...` 污染全局）
- `typeof null === 'object'` 不做判空防御（`typeof obj === 'object' && obj === null` 直接崩）
- `NaN` 比较（`NaN === NaN` 为 false，应用 `isNaN()` 或 `Number.isNaN()`）
- `parseInt` 不传第二参数（`parseInt('010')` 在不同环境下可能是 8 也可能是 10）
- `instanceof` 跨 iframe 或 context（不同的执行上下文 instanceof 会失败）
- `+` 运算符拼接数字和字符串（`1 + '2' = '12'`, `1 - '2' = -1` 行为不一致）
- 在循环中声明函数（每次迭代都创建一个新函数）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| `eval()` | 严重 | 任意代码执行，不可优化 |
| `new Function()` | 严重 | 等价于 eval |
| 修改内置原型 | 严重 | 破坏所有依赖该原型的代码 |
| 未处理 Promise | 严重 | 静默吞错，进程没有崩溃但状态全部不正常 |
| `with` | 严重 | 严格模式禁止，作用域不可预测 |
| 全局变量污染 | 高 | 多个脚本互踩命名空间 |
| 回调地狱 | 高 | 错误处理链断裂，逻辑不可读 |
| `var` 声明 | 高 | 变量提升导致意料之外的行为 |
| `==` 而非 `===` | 高 | 隐式类型转换，奇奇怪怪的比较结果 |
| `typeof null` 无防御 | 中 | 非常见但一旦命中直接崩 |
| `NaN` 比较 | 中 | 逻辑判断永远返回 false |
| `parseInt` 不传 radix | 中 | 老八进制 bug 可能仍然出现 |
| for...in 遍历数组 | 中 | 拿到原型属性，结果不可预期 |
| delete 数组元素 | 中 | 稀疏数组导致遍历和长度行为异常 |
| 循环中声明函数 | 中 | 不必要的内存分配 |
| arguments 对象 | 低 | 箭头函数里也没有 arguments |
| instanceof 跨 context | 低 | 特定场景才出现 |
| `+` 运算类型混乱 | 低 | 最常见的是 `1 + '2' = '12'` 但不会 crash |
