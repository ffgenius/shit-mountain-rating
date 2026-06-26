# C# 语言规范

C# 语言专项检查。

## 检测项

- `dynamic` 类型滥用（编译时类型检查完全绕过，运行时崩给你看）
- `var` 滥用（类型完全不明确时仍用 var，看代码猜不到类型）
- `#pragma warning disable` 禁用编译器警告（掩耳盗铃）
- 通用 `Exception` 捕获（吞掉所有异常不做区分处理）
- `Thread.Sleep` 同步阻塞（在主线程或请求线程中 Sleep）
- `lock(this)` 危险锁定（外部代码也能 lock 你的 this，死锁隐患）
- `Dispose` 未调用（实现了 IDisposable 但不用 using 也不手动 Dispose）
- `string` 在循环中用 `+=` 拼接（不用 StringBuilder，O(n²)）
- `null!` 使用（告诉编译器"这不可能为 null"，但实际可能是 null）
- `ConfigureAwait(false)` 滥用或不加（用错了导致上下文切换问题）
- `async void` 而非 `async Task`（异常无法被 await 方捕获）
- `Task.Result` / `Task.Wait()` 同步等待异步（死锁经典配方）
- `LINQ` 多次枚举（同一个 IEnumerable 遍历多次，每次触发新的数据库查询）
- `First()` 而非 `FirstOrDefault()`（集合空时直接抛异常）
- 用 `DateTime.Now` 而非 `DateTime.UtcNow`（时区问题）
- `struct` 中包含可变字段（值语义和引用语义混淆）
- 反射代替泛型或接口（用反射遍历属性比编译时多态慢几个数量级）
- `catch { throw; }` 无意义的捕获重抛（丢了原始堆栈信息）
- 用 `GC.Collect()` 手动触发垃圾回收（几乎总是错的）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| `async void` | 严重 | 异常无法被上游捕获，进程可能被未处理的异常杀死 |
| Task.Result 同步等待 | 严重 | 同步上下文死锁，在 ASP.NET 中经典故障 |
| lock(this) | 严重 | 任何人都能获取你的锁，死锁随时出现 |
| dynamic 滥用 | 严重 | 放弃编译时安全，运行时一个 Typo 就崩 |
| null! 使用 | 高 | 骗编译器，骗自己，运行时 NullReferenceException |
| 不用 using/Dispose | 高 | 数据库连接、文件句柄、Socket 泄漏 |
| Thread.Sleep | 高 | 阻塞线程池线程，降低吞吐量 |
| 循环中 += 字符串 | 高 | O(n²) 内存分配 |
| 通用 Exception 捕获 | 高 | 无差别吞异常 |
| catch throw | 高 | 原始堆栈丢失，排查问题困难 |
| LINQ 多次枚举 | 中 | 每次枚举都重新执行，结果不一致 |
| First() 而非 FirstOrDefault() | 中 | 不必要的异常 |
| var 滥用 | 中 | 代码可读性下降 |
| #pragma warning disable | 中 | 每个禁掉的警告都是一个潜在 bug |
| DateTime.Now | 中 | 行为依赖服务器时区 |
| struct 可变 | 中 | 值类型被修改的行为不符合直觉 |
| 反射代替泛型 | 中 | 慢，且 IDE 重构时不会更新字符串中的属性名 |
| ConfigureAwait 缺少/滥用 | 中 | 上下文捕获开销或不正确的行为 |
| GC.Collect() | 低 | 几乎永远不应该手动调用，除非你真的知道自己在做什么 |
