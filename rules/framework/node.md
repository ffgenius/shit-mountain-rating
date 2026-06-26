# Node.js 运行时规则

Node.js 运行时专项检查。

## 检测项

- 未处理 `uncaughtException`（进程随时可能因一个未捕获异常而 crash）
- 同步阻塞 I/O（`fs.readFileSync` 在请求处理中阻塞 Event Loop）
- HTTP 请求未设置超时（外部服务挂了，你的进程跟着一起挂起）
- Event Loop 阻塞（大数组同步循环、正则回溯、JSON.stringify 巨大对象）
- 内存泄漏（`setInterval` 不清理、EventEmitter 监听器累积、闭包持有大对象）
- `process.exit()` 直接退出（不关闭连接、不释放资源、不通知负载均衡）
- `require` 与 `import` 混用（CommonJS / ESM 互操作导致模块解析失败）
- 回调地狱（Callback 嵌套超过 3 层，不用 async/await 也不 promisify）
- 无优雅关闭（`SIGTERM` 到来时不处理，直接 kill 导致请求丢失）
- `process.env` 散落各处（配置没有集中管理，换环境像扫雷）
- `JSON.parse` 无 try-catch（接收到非法 JSON 直接崩）
- 用 `console.log` 当日志（无级别、无轮转、无结构化格式）
- 无健康检查端点（负载均衡不知道你的进程是否还活着）
- 无请求体大小限制（恶意 5MB JSON 直接打爆内存）
- 无 Rate Limiting（没有限流，一个脚本就能压垮服务）
- 无 Cluster / Worker（单进程跑满一个 CPU，其余核心在旁围观）
- 一次性加载全部数据到内存（读个大文件全量放变量里，无视流式处理）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| 未处理 uncaughtException | 严重 | 一个异常就能让整个进程 crash |
| 同步 I/O 在请求中 | 严重 | 所有请求排队等一个文件读完 |
| 回调地狱 | 严重 | 错误处理链断裂，任何一个环节出问题都是未捕获 |
| `JSON.parse` 无保护 | 严重 | 接收到非法 JSON 直接抛异常 crash |
| 无请求体大小限制 | 严重 | 恶意载荷打爆服务器 |
| 无超时 | 高 | 外部依赖挂了你也要扛着直到资源耗尽 |
| Event Loop 阻塞 | 高 | CPU 密集型操作阻塞所有 I/O |
| 内存泄漏 | 高 | 内存持续增长直到 OOM |
| `process.exit()` | 高 | 所有飞行中的请求全部丢失 |
| 一次性加载全量数据 | 高 | 大文件打爆内存 |
| 无优雅关闭 | 中 | 发版时丢失请求和数据 |
| process.env 散落 | 中 | 换了环境秘密就泄露，配置就失效 |
| console.log 打天下 | 中 | 排查问题没有结构化日志非常痛苦 |
| 无健康检查 | 中 | 挂了没人知道 |
| 无 Rate Limiting | 中 | 任何一个客户端都能把服务打穿 |
| 无 Cluster | 低 | 单核跑满其余围观，但也可能部署层已解决 |
| require/import 混用 | 低 | 具体情况取决于 Node 版本和项目设置 |
