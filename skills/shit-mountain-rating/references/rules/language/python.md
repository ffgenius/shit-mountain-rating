# Python 语言规范

Python 语言专项检查。

## 检测项

- `except:` 裸异常捕获（吞掉 KeyboardInterrupt、SystemExit 等所有异常）
- `except Exception: pass` 静默吞异常（出错了但假装一切正常）
- `global` 关键字滥用（共享可变状态，无法追踪修改来源）
- `eval()` / `exec()` 使用（任意代码执行）
- 可变默认参数（`def f(x=[])` 列表在所有调用之间共享）
- `type()` 代替 `isinstance()`（绕过多态和子类兼容）
- 循环内 `import`（每次循环都重新加载模块）
- `*args` / `**kwargs` 滥用且无类型提示（调用方不知道要传什么参数）
- `from module import *`（命名空间污染，不知道导入了什么）
- `__del__` 中做复杂逻辑（垃圾回收时机不确定，异常被忽略）
- `list` / `dict` 作为类属性（实例之间共享同一个列表/字典）
- `try-except` 包裹整个函数体（异常边界太大，真正的问题被掩盖）
- `is` 判断值和 `==` 混淆（`x is 5` 和 `x == 5` 在某些情况下不同）
- f-string 中调用函数（`f"Price: {get_price()}"` 有副作用时隐藏了函数调用）
- 用 `list.append` 在循环中构建列表而非列表推导式
- 函数参数类型注解和实际参数不匹配（`def foo(x: int): ...` 然后 `foo("123")`）
- `dict.get()` 而不检查 None（`d.get('key').upper()` 当 key 不存在时 attributeError）
- `open()` 不用上下文管理器（`f = open(); ... ; f.close()` 异常路径下文件句柄泄漏）
- `os.system()` 或 `subprocess` 传入 `shell=True`（命令注入风险）
- 过时的 `%` 格式化或 `.format()` 不统一（一个项目三种字符串格式化混用）

## 评分标准

| 问题 | 严重程度 | 说明 |
|------|----------|------|
| `eval()` / `exec()` | 严重 | 任意代码执行，不可信任的输入直接拿 shell |
| os.system / shell=True | 严重 | 命令注入风险 |
| 裸 except: | 严重 | 吞掉所有异常，包括你 Ctrl+C 想停程序都停不了 |
| 可变默认参数 | 严重 | 所有调用共享同一个 list/dict，数据积累在参数里 |
| list/dict 类属性 | 严重 | 所有实例共享，一个实例改了影响所有其他实例 |
| 文件不用 with | 严重 | 异常时文件不关闭，句柄泄漏 |
| from module import * | 高 | 命名空间完全不可控 |
| except: pass | 高 | 错误被吃掉，程序以不一致的状态继续运行 |
| open 不用上下文 | 高 | 资源泄漏 |
| global | 高 | 全局可变状态，任何函数都可能在任何时候修改 |
| __del__ 复杂逻辑 | 高 | 垃圾回收时机不确定，异常被静默忽略 |
| try 包裹整个函数 | 中 | 无法定位哪个调用出的问题 |
| dict.get() 不判断 None | 中 | 链式访问 AttributeError |
| 循环内 import | 中 | 频繁重新加载 |
| type() 代替 isinstance() | 中 | 破坏多态 |
| is 和 == 混用 | 中 | 值相等 vs 身份相同，不同场景用错 | 
| f-string 中有副作用函数 | 中 | 隐式函数调用难以追踪 |
| 不用列表推导式 | 低 | 性能差但功能正确 |
| 类型注解不匹配 | 低 | 不影响运行时但 IDE 提示错误 |
| 格式化方式不统一 | 低 | 只影响可读性 |
