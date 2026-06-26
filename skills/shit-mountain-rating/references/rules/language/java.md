# Java 语言规范

Java 语言专项检查。

## 检测项

- `Optional.get()` 不检查直接调用（没值就 NoSuchElementException）
- `static` 工具类滥用（StringUtil、DateUtil、NumberUtil 每个项目重复造一遍）
- `ThreadLocal` 未清理（线程池环境下 ThreadLocal 残留导致内存泄漏和脏数据）
- `synchronized` 滥用（整个方法加锁，并发度归零）
- 反射使用（绕过类型系统、破坏封装、慢、不可重构）
- `null` 返回而非 Optional（调用方不做 null 检查直接崩）
- `System.out.println` 代替日志框架（无级别、无格式、无轮转）
- `catch (Exception e)` 一把抓（吞掉所有异常无区分处理）
- `e.printStackTrace()` 在 catch 中（异常信息打到了没人看的标准错误流）
- `String` 用 `==` 比较（比的是引用而非值）
- 在循环中拼接 String（不用 StringBuilder，每次 `+` 都新 String）
- `float` / `double` 做货币运算（使用 `BigDecimal` 但也不提供 MathContext）
- 资源未 try-with-resources（手动关闭 InputStream/Connection，异常路径关不掉）
- `public static` 可变字段（全局可变状态，多线程灾难）
- `equals()` 重写但 `hashCode()` 遗漏（HashMap/HashSet 找不到元素）
- `finalize()` 使用（已废弃，执行时间不可控）
- `raw type` 使用（`List` 而非 `List<String>`，泛型白加）
- `Collections.synchronizedList` 的复合操作不加锁（遍历和判断-操作不在同一个 synchronized 块）
- `SimpleDateFormat` 作为静态字段（线程不安全）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| catch Exception 一把抓 | 严重 | 关键异常和预期异常被无差别吞掉 |
| 资源未 try-with-resources | 严重 | 异常路径下资源泄漏，连接池耗尽 |
| public static 可变字段 | 严重 | 全局可变状态 + 多线程 = 不可预测的 bug |
| equals 无 hashCode | 严重 | HashMap/HashSet 行为完全错误 |
| ThreadLocal 未清理 | 高 | 线程池复用 + ThreadLocal 残留 = 内存泄漏和脏数据 |
| Optional.get() 不检查 | 高 | 运行时异常直接炸 |
| null 返回 | 高 | NPE 是最常见的运行时异常 |
| 循环中拼接 String | 高 | O(n²) 内存分配 |
| 反射 | 高 | 慢、不安全、不可追踪、随便重构就炸 |
| float/double 货币 | 高 | 精度丢失导致的金额错误 |
| synchronized 整个方法 | 中 | 并发度归零 |
| static 工具类泛滥 | 中 | 难以测试、难以替换 |
| String == 比较 | 中 | 期望值比较却得到引用比较结果 |
| SimpleDateFormat 静态 | 中 | 线程不安全，格式化结果随机 |
| Collections.synchronizedList 裸用 | 中 | 复合操作不原子 |
| finalize | 中 | 已废弃，功能不可靠 |
| e.printStackTrace() | 中 | 输出到了没人看的地方 |
| System.out.println | 低 | 在生产环境无意义但在开发期高效 |
| raw type | 低 | 编译时警告不一定引发运行时错误 |
