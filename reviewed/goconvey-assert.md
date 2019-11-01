# 断言

goconvey的所有断言都用So()来调用。

主要包含：

- 通用比较
- 数值比较
- 集合判断
- 字符串判断
- 异常判断
- 类型比较
- 时间比较

## 自定义断言

    func should<do-something>(actual interface{},
      expected ...interface{}) string {
        if <some-important-condition-is-met(actual, expected)> {
            return ""   // empty string means the assertion passed
        }
        return "<some descriptive message detailing why the assertion failed...>"
    }

符合上面的格式，即可实现自定义断言
