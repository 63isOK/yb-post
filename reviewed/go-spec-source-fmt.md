# 源码显示格式

- 源代码是unicode文本,编码格式是utf-8
- 但并不是完全符合规范的,不管咋说，下面都会用字符来表示unicode的码点
  - 虽然`字符`并不是完全规范的用语
- 大小写是不同的字符
- 实现限制:
  - 为了兼容一些工具,不会允许显示NUL字符,也就是(U+0000)
  - 也可能会忽略(U+FEFF)开头的字符

## 字符

术语,这些术语会用在ebnf中:

- newline: U+000A
- unicode_char: 除了newline的任意字符
- unicode_letter: 字母类
- unicode_digit: 数字类/十进制数字

所以字符总共有两类：新行和字符(字符还分字母和数字)

Go中的字符还是遵循了unicode标准8.0的部分.

## 字母和数值

下划线属于字母

    letter        = unicode_letter | "_" .  
    decimal_digit = "0" … "9" .  
    binary_digit  = "0" | "1" .
    octal_digit   = "0" … "7" .  
    hex_digit     = "0" … "9" | "A" … "F" | "a" … "f" .

ebnf表示法,依次定义了字母/十进制数字/八进制/十六进制数字.
最新版本还添加了上面的二进制数字.
