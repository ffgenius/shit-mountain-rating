# Flask 框架规则

Flask 框架专项检查。

## 检测项

- 缺少应用工厂模式（`app = Flask(__name__)` 写在模块顶层，测试和环境切换困难）
- 路由中直接写业务逻辑（`@app.route` 里数据库查询、数据校验、模板渲染全挤在一起）
- 缺少蓝图拆分（所有路由在一个 app.py 里，上千行文件）
- `app.config` 硬编码（数据库密码、SECRET_KEY 直接写在代码里）
- 缺少统一错误处理器（未注册 `@app.errorhandler`，异常直接弹出 Werkzeug 调试页或 500 空白页）
- `request` 全局对象滥用（在非请求上下文中使用 request、g、session）
- 缺少数据库迁移管理（没有 Flask-Migrate / Alembic，手动改表结构）
- `debug=True` 在生产环境（暴露 Werkzeug 调试器和环境变量）
- 不配置请求钩子（before_request / after_request / teardown_request，无统一预处理）
- 自定义鉴权而非用 Flask-Login（自己写 session/cookie 处理，安全漏洞几率极高）
- SQLAlchemy Session 管理不当（session 不关闭不提交不回滚，连接池耗尽）
- 模板不继承 Jinja2 布局（每个模板都是完整 HTML，改了头部要改所有文件）
- 硬编码的 SECRET_KEY（`app.secret_key = 'dev-key-123'` 直接写在代码里）
- 不用蓝图但手动分割路由（`from other_file import routes; routes(app)` 自己造的协议）
- 依赖包散落不冻结（没有 requirements.txt 或 pyproject.toml，环境不可复现）
- 在生产用 Flask 内置服务器（`app.run()` 直接 serve 生产流量）
- 静态文件直接通过 Flask serve（应该交给 Nginx/CDN）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| debug=True 生产 | 严重 | 暴露代码和变量，可远程执行代码 |
| 硬编码 SECRET_KEY | 严重 | Session 可被伪造，所有用户可被劫持 |
| 生产用内置服务器 | 严重 | 单线程、无并发、无安全，随时崩溃 |
| SQLAlchemy Session 乱管理 | 严重 | 连接泄漏，数据库连接池耗尽 |
| 缺少应用工厂 | 高 | 测试、多环境部署、扩展初始化全部受阻 |
| 路由中直接业务逻辑 | 高 | 无分层，修改业务扫路由文件 |
| 缺少错误处理器 | 高 | 异常直接暴露给客户端，含内部信息 |
| 缺少数据库迁移 | 高 | 手工管理表结构，生产事故口袋 |
| app.config 硬编码 | 高 | 数据库密码、API Key 写死在代码里 |
| 自定义鉴权 | 高 | 安全漏洞率远高于成熟方案 |
| 缺少 Blueprint | 中 | 单文件上千行，模块边界消失 |
| request 全局滥用 | 中 | 测试和复用困难 |
| 模板不继承 | 中 | 修改布局工作量翻倍 |
| 无请求钩子 | 中 | 统一预处理逻辑（日志/鉴权）无法集中 |
| 静态文件 Flask serve | 中 | 浪费应用服务器资源 |
| 依赖未冻结 | 中 | 换环境/部署时版本不一致 |
