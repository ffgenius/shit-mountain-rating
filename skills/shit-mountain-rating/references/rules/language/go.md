# Go 语言规范

Go 语言专项检查。

## 检测项

- `panic()` 在业务代码中使用（panic 应该在 init 阶段，不应在请求处理中）
- `recover()` 滥用（不该 recover 的 panic 被默默吞掉）
- `context.Background()` 而非传入的 context（不传递取消信号和超时）
- `interface{}` 使用（1.18+ 应使用 `any` 或泛型，interface{} 丢失所有类型信息）
- Goroutine 泄漏（goroutine 启动了但没有退出机制）
- `defer` 缺失导致资源泄漏（打开文件/连接后没有 defer Close）
- 错误被忽略（`_ = err` 或直接不接收 error 返回值）
- `go func()` 闭包捕获循环变量（Go 1.21 之前经典 bug，1.22 修复但老代码可能还在）
- 无缓冲 channel 滥用（send 和 receive 必同步导致死锁）
- `sync.Mutex` 被拷贝（`go vet` 警告但很多人忽略）
- 在 `for range` 中修改被遍历的 slice（循环内 append 或删除元素）
- `map` 并发读写（不加锁的 map 被多个 goroutine 同时读写直接 fatal）
- `time.Tick` 而非 `time.NewTicker`（Tick 没有 Stop 方法，GC 都回收不了）
- `string(byteSlice)` 把 []byte 转 string 用在热路径（每次转换都拷贝）
- 返回值命名但没有意义（`func foo() (err error)` 然后 return 前手动设 err）
- `init()` 函数过多且做重活（init 阻塞包初始化，且顺序依赖 fragile）
- `defer` 在循环中使用（延迟到函数返回而非每圈执行，资源只增不减）
- `nil` interface 判断陷阱（interface 的 nil 和底层值的 nil 不是一回事）
- `ioutil` 包使用（Go 1.16+ 已废弃，应在 io/os 包中）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| map 并发读写 | 严重 | 不加锁直接 fatal，进程 crash |
| panic 在业务代码 | 严重 | 一个 panic 没 recover 整个 goroutine 挂 |
| goroutine 泄漏 | 严重 | goroutine 只增不减，内存泄漏到 OOM |
| defer 缺失 | 严重 | 文件句柄/连接泄漏，耗尽文件描述符 |
| 错误被忽略 | 严重 | 错误路径被丢弃，调用方以为一切正常 |
| sync.Mutex 拷贝 | 高 | 锁失效，go vet 会报但运行时不崩溃只形同虚设 |
| 无缓冲 channel 死锁 | 高 | 发送方和接收方互相等，goroutine 卡死 |
| for range 中修改 slice | 高 | 遍历行为不可预测，可能跳过或重复元素 |
| context.Background() | 高 | 丢失取消信号和超时，调用链不可控 |
| time.Tick | 中 | 内存泄漏，且没法停止 |
| go func 闭包捕获循环变量 | 中 | 1.22 之前经典 bug，所有 goroutine 看到相同值 |
| defer 在循环中 | 中 | 资源累积未释放 |
| interface{} | 中 | 类型信息丢失，断言崩 |
| 循环中 string([]byte) | 中 | 不必要的内存分配 |
| nil interface 陷阱 | 中 | 运行时 nil 判断失效 |
| init 函数重活 | 低 | 包加载顺序脆弱 |
| 返回值命名但不简化代码 | 低 | 增加理解成本 |
| ioutil 废弃 | 低 | 但功能仍然正常 |
