# 符号

语法是ebnf(计算机用语)来描述的,
ebnf是扩展的巴科斯范式,现代编程语言都是这种

具体的语法是:

    Production  = production_name "=" [ Expression ] "." .
    Expression  = Alternative { "|" Alternative } .
    Alternative = Term { Term } .
    Term  = production_name | token [ "…" token ] | Group | Option | Repetition .
    Group       = "(" Expression ")" .
    Option      = "[" Expression "]" .
    Repetition  = "{" Expression "}" .

一般新的编程语言都使用ebnf来定义语法规则,上面就是go的

分析：

- production就是类似  a=b这种包含左右的
- | 表示多选一
- {}表示重复  0次或n次
- () 表示分组
- `[]` 表示可选 0次或1次
- 分组 group 的形式是 (表达式)
- 可选 option 的形式是 `[表达式]`
- 重复 repetition 的形式是 {表达式}
- a...b 表示可选，a到b范围之内都可以表示，只选一个
  - 注意,这里的三个点不是小数点,而是代表省略号
  - 在spec中未做特殊说明的,...表示的省略号

小写的name(这里指`production_name`)用于标识一个字面量token，
字面量token一般被双引号或反引号包裹。

... 也可用于其他地方的表示，eg：枚举/不进一步展开的代码段,
另外...不是token。

spec中的token是有特殊意义的,下文会提到
