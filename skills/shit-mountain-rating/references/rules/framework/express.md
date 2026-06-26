# Express 框架规则

Express.js 框架专项检查。

## 检测项

- 缺少错误处理中间件（未注册 `(err, req, res, next)` 中间件，异常直抛客户端）
- 路由逻辑过重（一个 `app.post('/order')` 里写 200 行，包含校验、业务、数据库）
- 中间件顺序不当（错误处理中间件不放在最后、鉴权放在路由之后）
- 缺少输入验证（`req.body.price` 直接进数据库，不校验类型和范围）
- 缺少 CORS 配置（跨域请求要么全拒要么全放）
- 未使用 `helmet`（缺少安全头：CSP、X-Frame-Options 等）
- 回调地狱（不用 async/await，`db.query(sql, (err, res1) => { db.query(...) })`）
- 无请求日志（没有 morgan 或类似中间件，出问题不知谁来访问了谁）
- 无响应压缩（Gzip/Brotli 不开，大 JSON 响应白白多发几倍流量）
- 无 `body-parser` 大小限制（`app.use(express.json())` 不带 `limit` 参数）
- 路由处理器中直接操作数据库（Controller 还没出生，Model 还没抽象）
- 无 Request ID 追踪（一个请求穿过多个服务，日志里无法关联）
- `res.json()` 返回敏感字段（密码哈希、内部 ID、Token 直接返回给客户端）
- 无 Rate Limiting（没有 `express-rate-limit`，用户密码可以被暴力枚举）
- req/res 对象传入 Service 层（把 HTTP 层的东西漏到了业务层）
- 静态文件不走 `express.static`（手动 `fs.readFileSync` 读取静态文件）
- 同步和异步错误处理混用（Promise 的错误不会被 Express 自动捕获）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| 缺少错误处理中间件 | 严重 | 无统一兜底，未捕获异常会导致连接挂起或泄露内部信息 |
| 无输入验证 | 严重 | 请求体未校验直接入库，安全隐患 |
| 同步/异步错误混用 | 严重 | Express 4 不捕获 Promise rejection，进程可能 crash |
| 无 body 大小限制 | 严重 | 恶意大 JSON 可耗尽内存 |
| 路由直接操作数据库 | 严重 | 没有分层，路由 200 行，改个字段扫全文件 |
| res.json 泄露敏感字段 | 严重 | 密码哈希、Token 返回给客户端 |
| 中间件顺序不当 | 高 | 鉴权/限流可能被绕过 |
| req/res 传入 Service | 高 | HTTP 层泄漏到业务层，复用和测试都困难 |
| 回调地狱 | 高 | 错误处理链断裂，代码不可读 |
| 无 CORS | 中 | 要么全拒绝要么全接受，都不对 |
| 无 helmet | 中 | 少了一整套安全 HTTP 头 |
| 无请求日志 | 中 | 出了问题只能靠猜 |
| 无压缩 | 中 | 大 JSON 白白多传几倍，但 CDN 可能已覆盖 |
| 无 Rate Limiting | 中 | 暴力枚举、爬虫、DDoS 均可乘 |
| 无 Request ID | 低 | 日志无法关联，排查问题时多一步手动关联 |
| 静态文件不用 express.static | 低 | 但可能 Nginx 已经处理了 |
