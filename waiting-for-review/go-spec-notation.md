# 符号

语法是ebnf(计算机用语),扩展的巴科斯范式,现代编程语言都是这种

具体的语法是:

    Production  = production_name "=" [ Expression ] "." .
    Expression  = Alternative { "|" Alternative } .
    Alternative = Term { Term } .
    Term  = production_name | token [ "…" token ] | Group | Option | Repetition .
    Group       = "(" Expression ")" .
    Option      = "[" Expression "]" .
    Repetition  = "{" Expression "}" .

一般新的编程语言都使用ebnf来定义语法规则,上面就是go的
