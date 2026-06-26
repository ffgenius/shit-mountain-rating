# Django 框架规则

Django 框架专项检查。

## 检测项

- Fat Models（一个 Model 类里塞了 30 个方法和 20 个属性）
- God Views（视图函数里 200 行业务逻辑、ORM 查询、模板渲染混在一起）
- N+1 查询（`select_related` / `prefetch_related` 缺失，列表页触发几百次 SQL）
- `DEBUG = True` 在生产环境（堆栈、SQL、配置一览无余）
- 缺少数据库索引（高频查询的字段没索引，SQL 全表扫描）
- `null=True, blank=True` 滥用（分不清 null 和 blank 的语义区别，无脑加两行）
- Signal 滥用（业务逻辑全在 Signal 里，隐式触发，chain 追踪不到）
- Settings 文件混乱（一个 settings.py 同时管开发、测试、生产，靠注释切换）
- View 中直接写原生 SQL（`connection.cursor().execute("SELECT ...")` 绕过 ORM）
- 自定义模板标签做数据库查询（模板渲染时偷偷查数据库，渲染速度随数据增长下降）
- 中间件包含业务逻辑（在 Middleware 里改了请求响应的业务数据）
- 文件上传不校验类型和大小（用户可上传任意文件、任意大小）
- 不用 `ModelForm`（表单验证写了 100 行代码，ModelForm 三行就能搞定）
- URL 硬编码（模板里 `/user/{{ id }}/detail/` 而不是 `{% url 'user-detail' id %}`）
- 不使用 Django 的缓存框架（同样的查询每次请求都打一次数据库）
- 管理命令中包含业务逻辑（`manage.py xxx` 里直接操作 ORM 处理复杂业务）
- 不在事务中做多表操作（关联数据的一个表写成功了另一个表写失败了没有回滚）
- 不继承 Django 的 AbstractUser 而自己写用户模型（放弃权限系统，安全问题从头造轮子）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| DEBUG=True 生产 | 严重 | 泄露堆栈、SQL、配置变量、SECRET_KEY |
| View 中直接写 SQL | 严重 | SQL 注入风险 + ORM 体系被完全绕过 |
| 文件上传不校验 | 严重 | 任意文件上传，远程代码执行风险 |
| 无事务多表操作 | 严重 | 数据不一致，一半写入成功一半失败 |
| N+1 查询 | 严重 | 循环内 ORM 查询，几千次 SQL |
| 硬编码 URL | 高 | 重构 URL 结构时全站修改 |
| Fat Models | 高 | 一个类 30+ 方法，模型层垄断所有逻辑 |
| God Views | 高 | View 中业务逻辑不加提取 |
| 中间件含业务逻辑 | 高 | 业务逻辑不应该在请求处理框架中 |
| Signal 滥用 | 高 | 隐式触发链不可追踪，出 bug 找不到根源 |
| Settings 混乱 | 高 | 换环境靠手工改代码注释，迟早出事故 |
| 模板标签查数据库 | 中 | 渲染速度取决于数据量，不可控 |
| 不用 ModelForm | 中 | 手写验证重复代码 |
| 管理命令含业务 | 中 | 业务逻辑不该绑定 CLI |
| 不用缓存 | 中 | 高频查询反复打库 |
| null/blank 滥用 | 中 | 数据库校验和表单校验语义混淆 |
| 缺少索引 | 中 | 查询慢，全表扫描 |
| 不用 AbstractUser | 中 | 放弃 Django 成熟权限系统 |
