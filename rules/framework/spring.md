# Spring 框架规则

Spring Boot / Spring 框架专项检查。

## 检测项

- 巨型 Service 类（>800 行，`OrderServiceImpl` 里有支付、物流、发票逻辑）
- Mapper / DTO 膨胀（一个 Entity 对应七八个 DTO，转换逻辑比业务还复杂）
- Bean 循环依赖（A → B → C → A，Spring Boot 2.6+ 直接禁止）
- `@Transactional` 滥用（每个方法都加，不管需不需要只读事务）
- `@Autowired` 字段注入（私有字段上直接注解，测试时无法注入 Mock）
- `@Lazy` 掩盖循环依赖（用懒加载绕过循环依赖而不重构代码）
- JPA N+1 查询（关联查询不用 `@EntityGraph` 或 `JOIN FETCH`）
- 缺少 `@Valid` / `@Validated` 验证（Controller 入参裸奔）
- `catch (Exception e)` 在 Controller 里（吞掉所有异常，返回"系统错误"）
- 静态工具类泛滥（xxUtil / xxHelper 到处都是，纯函数不归容器管理）
- 不用分页查询全量数据（`findAll()` 直接返回几万条）
- `EAGER` 加载所有关联（打开一个订单，加载了用户、商品、物流、评价）
- JPA 和 JDBC 混用同一事务（同一业务逻辑里 ORM 和裸 SQL 搞混）
- `@Scheduled` 无 `fixedDelay` 调优（上一个任务没跑完下一个又开始了）
- `@Query` 中用字符串拼 SQL（把 JPA 当 JDBC 用）
- Controller 中写业务逻辑（Service 变成了贫血层，Controller 里几百行代码）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| 循环依赖 | 严重 | Spring Boot 2.6+ 默认禁止，启动直接报错 |
| catch Exception | 严重 | 吞掉所有异常且无任何上下文 |
| N+1 查询 | 严重 | 列表页几百条数据触发几千次 SQL |
| 全量查询无分页 | 严重 | 一次请求查几万条数据，OOM |
| EAGER 加载全部关联 | 严重 | 查一个实体连带全家，SQL Join 地狱 |
| `@Query` 拼 SQL | 严重 | SQL 注入 + 可读性灾难 |
| 巨型 Service | 高 | 超过 800 行，修改一处可能波及整个系统 |
| `@Autowired` 字段注入 | 高 | 不可测试、不可不变、隐藏依赖 |
| `@Transactional` 滥用 | 高 | 只读操作加写事务，长事务锁表 |
| `@Lazy` 掩盖循环 | 高 | 掩耳盗铃，迟早出更大的架构问题 |
| 业务逻辑在 Controller | 高 | Controller 变上帝类，Service 贫血 |
| 静态工具类泛滥 | 中 | 难以测试、难以替换、难以追踪 |
| Mapper/DTO 膨胀 | 中 | 转换层比业务层还厚 |
| 混用 JPA/JDBC | 中 | 事务边界模糊，一致性难保 |
| 无 `@Valid` | 中 | 未校验的数据直接入 Service 层 |
| 定时任务无锁 | 中 | 重复执行导致数据错误 |
