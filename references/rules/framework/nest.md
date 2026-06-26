# NestJS 框架规则

NestJS 框架专项检查。

## 检测项

- 循环依赖注入（Module A → B → C → A，启动直接炸）
- 巨型 Module（一个 Module 注册了几十个 Controller 和 Provider）
- Service God Object（单个 Service 包含了各种不相关的业务逻辑）
- Decorator 滥用（自定义 Decorator 过多、逻辑藏在 Decorator 里难以追踪）
- Provider 层级混乱（请求作用域和单例混用，注入时机不可预测）
- 缺少 DTO 验证（未使用 `class-validator`，随便什么数据都能进 Controller）
- Guard / Interceptor 逻辑过重（鉴权和日志拦截器里写了几百行业务逻辑）
- 直接使用 `@Res()` response 对象（绕过 NestJS 的响应拦截和异常过滤器）
- Service 中拼接原始 SQL（在 NestJS 里写 `SELECT * FROM users WHERE ${name}`）
- 异常过滤器不统一（每个 Controller 自己 try-catch，报错格式五花八门）
- 用 `console.log` 代替 Logger（生产环境日志无法分级、无法持久化）
- `onModuleInit` 中做异步操作但不阻塞启动（模块初始化未完成就接受请求）
- 把业务逻辑放在 Controller 中（Controller 比 Service 还厚）
- 不提取配置到 ConfigService（process.env 散布在每个 Service 里）
- `@Cron()` 定时任务无防抖和锁（定时任务并发执行导致数据错乱）
- Module 导入依赖过多（一个 Module imports 了 20 个 Module）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| 循环依赖 | 严重 | 启动失败，或运行时拿到 undefined 的依赖 |
| 拼接原始 SQL | 严重 | SQL 注入风险，且 ORM 的优势全部丢失 |
| 无 DTO 验证 | 严重 | 任何数据都能直接到 Controller，安全问题 |
| 直接操作 @Res() | 高 | 拦截器、异常过滤器、序列化全部被绕过 |
| God Object Service | 高 | 一个 Service 包含多种不相关业务逻辑 |
| 巨型 Module | 高 | 模块边界消失，依赖关系不可控 |
| 业务逻辑在 Controller | 高 | Controller 变胖，Service 被架空 |
| 异常过滤器不统一 | 高 | 报错格式五花八门，前端对接噩梦 |
| console.log 打天下 | 中 | 生产环境无日志分级，排查问题靠猜 |
| onModuleInit 不阻塞 | 中 | 启动后模块可能未初始化完成 |
| Decorator 滥用 | 中 | 自定义 Decorator 过多，行为藏得太深 |
| Provider 混乱 | 中 | 作用域配置不当导致注入失败或内存泄漏 |
| 不提取配置 | 中 | process.env 散布，换个环境就炸 |
| Guard/Interceptor 过重 | 中 | 鉴权中间件包含了业务规则 |
| 定时任务无锁 | 中 | 并发执行可能导致数据不一致 |
