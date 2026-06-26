# TypeScript 语言规范

TypeScript 语言专项检查。

## 检测项

- `any` 类型使用（用 `any` 放弃了所有类型安全保障）
- `as any` 强制类型断言（"别管了，编译器你听我的"）
- `@ts-ignore` 注释（单行绕过类型检查，每处都是定时炸弹）
- `@ts-nocheck` 文件级别跳过检查（整个文件退回 JavaScript）
- 隐式 `any`（函数参数无类型标注，返回值无类型标注）
- `namespace` 使用（应使用 ES module 的 import/export）
- `enum` 滥用（数字枚举在运行时不可预测，字符串联合更好）
- `as` 类型断言取代类型守卫（不用 `instanceof`、`typeof`，直接断言）
- `!` 非空断言滥用（`obj!.prop!.nested` 比 `any` 好不到哪去）
- `return` 类型不匹配（声明了返回 `T` 实际返回 `T | undefined`）
- `interface` 与 `type` 混用无规则（同一个结构有的用 interface 有的用 type）
- `public` 关键字冗余（TypeScript 默认就是 public）
- 用 `Object` / `Function` / `Number` 等包装类型代替原始类型（`String` 和 `string` 是不同的）
- 类型收窄遗漏（if 分支已经排除了 null 还加 `?`）
- 泛型参数默认值缺失（不传参数时直接变成 `unknown`）
- `readonly` 不用（本该不可变的对象到处被改）
- `strictNullChecks` 关闭（开启严格模式的 TypeScript 才叫 TypeScript）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| `@ts-nocheck` | 严重 | 整个文件放弃类型检查，TypeScript 只拿来写注解 |
| `as any` | 严重 | 强制绕过类型系统，等于在类型安全的墙上开洞 |
| `@ts-ignore` | 高 | 每处一个隐患，掩盖了本应修复的类型错误 |
| 隐式 `any` | 高 | 函数签名没有类型，调用方不知道传什么 |
| `!` 非空断言滥用 | 高 | 告诉编译器"这里一定有值"，但万一没有就是运行时崩 |
| strictNullChecks 关闭 | 高 | null/undefined 到处乱窜，无法区分必须有值和可能无值 |
| `any` 类型 | 中 | 偶尔一两个可以理解，超过 5 个说明类型设计有问题 |
| `as` 代替类型守卫 | 中 | 用断言取代正确的类型收窄 |
| 包装类型 | 中 | `String` 和 `string` 行为不同，引发难以追踪的 bug |
| 泛型无默认值 | 中 | 调用时直接变成 unknown |
| return 类型不匹配 | 中 | 可能遗漏边界情况的返回值 |
| interface/type 混用 | 低 | 代码风格问题，可以用 lint 统一 |
| `namespace` | 低 | 过时但有历史遗留的合理场景 |
| enum 滥用 | 低 | 字符串联合类型更优，但 enum 有 const enum 编译优化 |
| public 冗余 | 低 | 不影响功能但增加噪音 |
| readonly 缺失 | 低 | 代码意图不明确 |
