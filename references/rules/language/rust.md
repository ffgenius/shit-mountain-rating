# Rust 语言规范

Rust 语言专项检查。

## 检测项

- `unwrap()` 在非原型/测试代码中使用（生产代码中 panic 是不可接受的）
- `expect()` 滥用（expect 的语义是"这里不可能出错"，但很多人当 unwrap 用）
- `unsafe` 块（绕过了 Rust 的所有安全保证，需要逐一审查）
- `Rc<RefCell<T>>` 模式（内部可变性逃逸，绕过借用检查器）
- `clone()` 滥用（该用借用或 take 的地方无脑 clone）
- `.to_string()` 滥用（该用 `&str` 参数的地方先 to_string 再传引用）
- 大量使用 `mut`（应该是不可变绑定的用了可变绑定）
- `Box::leak` 或 `mem::forget`（故意泄漏内存）
- 在 match 中用 `_ => {}` 吞掉所有其他分支（新增枚举变体时编译器不会报遗漏）
- `as` 进行有损类型转换（`u64 as usize` 在 32 位平台截断）
- 循环内使用 `collect()`（每次循环都分配新 Vec）
- `println!` 代替日志（生产代码用 println 输出调试信息）
- `panic!` 而非 `Result`（库代码中 panic 意味着调用方无法恢复）
- 过多 `Arc` 包装（能传引用却用 Arc 计数）
- `static mut` 全局可变状态（线程安全全靠自觉）
- 不用 `#[derive]` 手写 trait 实现（手写 Debug、Clone、PartialEq）
- 泛型函数中 String 参数该用 `&str` 或 `AsRef<str>` 收窄（不必要的字符串拷贝）
- `vec![]` 在循环中被频繁 push（应用 `collect()` 或者 `with_capacity`）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| `unsafe` 块 | 严重 | 每一处都是潜在的 UB，必须逐一审查确认安全 |
| `static mut` | 严重 | 全局可变状态 + 线程安全无保证 |
| `panic!` 在库代码 | 严重 | 库代码不应决定进程生死 |
| `unwrap`/`expect` 在生产代码 | 高 | 一个 unwrap 失败整个进程 terminate |
| `as` 有损转换 | 高 | 截断错误难以追踪，32 位平台静默出 bug |
| `_ => {}` 吞噬分支 | 高 | 新增枚举变体时编译器不再报 cover 不全的编译错误 |
| `clone()` 滥用 | 中 | 不必要的堆分配和拷贝 |
| `.to_string()` 滥用 | 中 | 应该传 &str 的地方先造个 String 再借走 |
| Box::leak / mem::forget | 中 | 内存泄漏，但安全 Rust 本来也不保证不泄漏 |
| 不用 #[derive] | 中 | 手写带来的 bug 比 derive 多 |
| 循环内 collect | 中 | 每圈分配新堆内存 |
| println! 代日志 | 中 | 生产环境日志不可控 |
| 过多 Arc | 中 | 能用引用的情况用 Arc 是多此一举 |
| 大量 mut | 低 | 不可变优先，但取决于场景 |
| Rc<RefCell> | 低 | 有正当用途（如 GUI），但滥用表明架构问题 |
