# Angular 框架规则

Angular 框架专项检查。

## 检测项

- 巨型 Component（>300 行且涵盖 UI + 业务逻辑 + 数据处理）
- Service 职责不清（UserService 里写订单逻辑，OrderService 里改用户信息）
- 模板中使用 `any` 类型（`*ngFor="let item of data$ | async as any"`）
- `ngOnInit` 中复杂逻辑（初始化中塞了大量异步和业务判断）
- 缺少 `OnPush` Change Detection（默认策略导致全树脏检查）
- 未 unsubscribe 的 Observable（`ngOnDestroy` 里空无一物，内存泄漏）
- Module 结构混乱（一个 Module 包含几十个 Component 和 Service）
- `ngOnChanges` 滥用（手动对比前后值，写成 if-else 地狱）
- @Input setter 中有副作用（setter 里发 HTTP 请求或改其他状态）
- 直接操作 DOM（`document.querySelector` / `element.nativeElement.style.xxx`）
- `ngFor` 没有 `trackBy`（大列表每次全部重建 DOM）
- 混合模板驱动表单和响应式表单（同一个 form 里两种方式共存）
- 在 Component 中 subscribe 而非使用 `async` 管道（手动管理订阅生命周期）
- `ViewChild` 在 `ngAfterViewInit` 之前访问（undefined 引用）
- 构造器中做重活（`constructor` 里调 HTTP 或做复杂计算）
- Pipe 设为 `pure: false` 无必要（每次变更检测都重新执行 Pipe）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| 未 unsubscribe | 严重 | Observable 未取消订阅，路由切换后旧组件仍然响应 |
| 直接操作 DOM | 严重 | 绕过 Angular 的渲染机制，变更检测无法知晓 DOM 变化 |
| 构造器做重活 | 严重 | 对象创建被阻塞，DI 容器初始化卡死 |
| 巨型 Component | 高 | 一个文件承担 UI + 逻辑 + 数据处理 |
| 缺少 OnPush | 高 | 默认策略对每个组件做脏检查，列表页卡死 |
| 缺少 trackBy | 高 | 大列表每次重建 DOM |
| Service 职责不清 | 高 | 业务逻辑散落，同一个概念在多个 Service 中重复 |
| 表单混用 | 中 | 同一表单两套 API 共存，校验逻辑分裂 |
| @Input setter 副作用 | 中 | 输入属性变更触发 HTTP 请求等副作用 |
| 模板 any 类型 | 中 | 失去类型安全保障 |
| Module 混乱 | 中 | 编译慢、依赖关系难以梳理 |
| 不用 async 管道 | 中 | 手动管理订阅生命周期容易遗漏 |
| ngOnChanges 滥用 | 低 | 简单逻辑写了上百行 |
| pure: false 滥用 | 低 | 性能消耗，但在小列表中影响有限 |
