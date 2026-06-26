# 错误处理评分规则

分析错误处理质量。

## 检测项

- Empty Catch（异常被吞掉，无任何处理或日志）
- Promise/异步未处理（未 catch 的 Promise、未处理的 rejection）
- throw 非 Error 对象（throw String / Number 导致丢失调用栈）
- Silent Error（错误被静默处理，调用方不知道出错了）
- 缺少重试机制（网络请求/数据库操作/文件 I/O 无重试或退避）
- catch 过于宽泛（catch Exception / catch (e: any) 不做区分）
- 错误信息无上下文（throw new Error("出错了") 无定位信息）
- 资源泄漏风险（错误路径未释放文件句柄/数据库连接/锁）

## 阈值定义

| 指标 | 警告 | 严重 |
|------|------|------|
| Empty Catch | — | 每处即严重 |
| 未处理 Promise | — | 每处即严重 |
| throw 非 Error | — | 每处即严重 |
| Silent Error | 1 处 | 2 处以上 |
| 缺重试 | 非关键路径 | 关键路径（支付/认证） |
| catch 过于宽泛 | 非关键路径 | 关键路径 |
| 错误无上下文 | 3-5 处 | >5 处 |
| 资源泄漏 | 1 处 | 2 处以上 |

## 评分公式

```
分数 = 100 - (Empty Catch × 20) - (未处理Promise × 20) - (throw非Error × 15) - (Silent Error严重 × 15) - (缺重试严重 × 10) - (catch宽泛严重 × 8) - (错误无上下文严重 × 5) - (资源泄漏严重 × 10) - (警告 × 3)
```

最低 0 分。Empty Catch 和未处理 Promise 各扣 20 分——生产事故的直接根源。

## 检测方式

- 扫描空 catch 块
- 检测未链式 catch 或 try-catch 的 await 表达式
- 检测 throw 的非 Error 类型
- 分析 catch 块内日志/上报逻辑
- 检测 http/db/io 调用周围的重试逻辑
- 错误路径中资源释放检查
