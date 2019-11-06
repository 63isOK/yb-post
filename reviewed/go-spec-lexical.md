# 词法的组成元素

## 注释

注释作为程序的文档,有两种写法

    行注释 //
    块注释 /*  */

注释不能出现在rune/字符串字面量/或注释中。

不包含换行的注释可理解为一个空格，其他注释可理解为一个新行

## tokens

tokens组成了go语言的词汇,主要有4类:

- 标识符
- 关键字
- 操作符和标点
- 字面量

空白由下面几个部分组成

- 空格
- tab
- 回车
- 换行

空白一般会被忽略,只有一种情况是例外：将两个token分开时不会被忽略。

换行/文件尾都可能会触发;的插入

## 分号

;的出现,意味着一个productions的结束,productions是ebnf的一个概念

go会在下面两种情况下,忽略;的输入:

- 输入被分解为token,多个token会组成一条go语句,
这个go语句被称为(token流),如果在token流中,
找到了一个final token,会自动添加一个分号;
final token可以是下面几个:
  - 标识符
  - int/float/复数/rune/string 字面量
  - 关键字:break/continue/fallthrough/return
  - ++ -- ) ] }
- 复杂语句占一行,在 ) } 前面,可以忽略掉分号

## 标识符 identifiers

标识符是程序实体的一个名字，eg：变量名/类型名。
标识符由一个或多个字母数字组成的序列,第一个字符需要是字母

ebnf的定义是:

    identifier = letter { letter | unicode_digit } .

identifiers可以由下划线开头, 前面也说了,下划线属于字母

Go已经预定义了一些标识符：类型/常量/零值/函数,都是些内建标识符。

## 关键字

    break        default      func         interface    select
    case         defer        go           map          struct
    chan         else         goto         package      switch
    const        fallthrough  if           range        type
    continue     for          import       return       var

关键字目前有25个,都是比较常用的

## 操作符和标点

    +    &     +=    &=     &&    ==    !=    (    )
    -    |     -=    |=     ||    <     <=    [    ]
    *    ^     \*=    ^=     <-    >     >=    {    }
    /    <<    /=    <<=    ++    =     :=    ,    ;
    %    >>    %=    >>=    --    !     ...   .    :
         &^          &^=

这上面都是一些常见的

## 整数字面量

整数字面量是表示整形常量的数字序列。

整数可以有个可选的前缀：

- 0b/0B 二进制
- 0/0o/0O 八进制
- 0x/0X 十六进制

十六进制中，a-f A-F 被认为是10-15

    int_lit        = decimal_lit | binary_lit | octal_lit | hex_lit .
    decimal_lit    = "0" | ( "1" … "9" ) [ [ "_" ] decimal_digits ] .
    binary_lit     = "0" ( "b" | "B" ) [ "_" ] binary_digits .
    octal_lit      = "0" [ "o" | "O" ] [ "_" ] octal_digits .
    hex_lit        = "0" ( "x" | "X" ) [ "_" ] hex_digits .

    decimal_digits = decimal_digit { [ "_" ] decimal_digit } .
    binary_digits  = binary_digit { [ "_" ] binary_digit } .
    octal_digits   = octal_digit { [ "_" ] octal_digit } .
    hex_digits     = hex_digit { [ "_" ] hex_digit } .

分析：

- 先定义字面量，再定义2/8/10/16进制的数字
- 里面的下划线可出现在进制后面或连续数字之间，但不影响具体的数值

## 浮点字面量

是一个浮点常量的10/16进制表示。

10进制浮点字面量包含：

- 整数部分
- 小数点
- 小数部分
- 指数部分(e/E开头，表示10的多少次方)

整数部分和小数部分都可以被忽略;小数点或指数部分也可能被忽略

16进制浮点字面量包含：

- 前缀 0x/0X
- 整数
- 小数点
- 小数部分
- 指数部分(p/P开头，十进制，表示2的多少次方)

    float_lit         = decimal_float_lit | hex_float_lit .

    decimal_float_lit = decimal_digits "." [ decimal_digits ]
                        [ decimal_exponent ] |
                        decimal_digits decimal_exponent |
                        "." decimal_digits [ decimal_exponent ] .
    decimal_exponent  = ( "e" | "E" ) [ "+" | "-" ] decimal_digits .

    hex_float_lit     = "0" ( "x" | "X" ) hex_mantissa hex_exponent .
    hex_mantissa      = [ "_" ] hex_digits "." [ hex_digits ] |
                        [ "_" ] hex_digits |
                        "." hex_digits .
    hex_exponent      = ( "p" | "P" ) [ "+" | "-" ] decimal_digits .

浮点字面量不存在8进制，所以 0123是8进制，0123.3 就是10进制123.3

## 复数字面量

复数字面量表示复数常量，由整数字面量或浮点字面量组成，外加小写的i

为了保持兼容，0开头的复数字面量，和浮点字面类似，都是指十进制的，
而不是走整数字面量的。

    imaginary_lit = (decimal_digits | int_lit | float_lit) "i" .

## rune字面量

rune字面量用rune常量表示,也就是标识unicode码点的整数。

rune就是单引号包围的一个或多个字符,eg: '\n' 'x',
在单引号内,不能出现两种字符:换行和未转义的单引号.
一个单引号包裹的字符,就是字符的unicode值,和 \多个字符 的写法是同一个东西.

为啥单引号内不能出现换行，因为 go里的字符 = unicode字符 + 换行

单引号包裹的单字符,是最常见的写法,go源码是unicode字符集,而且是utf-8编码,
多个utf-8编码的字节,可能表示一个整数.

如果值在ascii文档中,表示一个整数有4种写法:

- \x 后跟2个十六进制数 eg: \x3f
- \u 后跟4个十六进制数 eg: \u3f2c
- \U 后跟8个十六进制数 eg: \U2d1a3c4d
- \  后跟3个八进制数 eg: \632

上面4中都可以表示一个整数，区别是她们的范围有些不一样。

除了上面的，\后面还可以跟一些特殊字符，eg：不可见的控制字符等，
除了上面几种，\后面出现其他字符都是非法的。

还要考虑\后面可能出现的一些转义字符

    rune_lit         = "'" ( unicode_value | byte_value ) "'" .
    unicode_value = unicode_char | little_u_value | big_u_value | escaped_char .
    byte_value       = octal_byte_value | hex_byte_value .
    octal_byte_value = `\` octal_digit octal_digit octal_digit .
    hex_byte_value   = `\` "x" hex_digit hex_digit .
    little_u_value   = `\` "u" hex_digit hex_digit hex_digit hex_digit .
    big_u_value      = `\` "U" hex_digit hex_digit hex_digit hex_digit
                               hex_digit hex_digit hex_digit hex_digit .
    escaped_char = `\` ("a"|"b"|"f"|"n"|"r"|"t"|"v"|`\`|"'"|`"`) . 

## string字面量

就是字符串常量,rune表示的单个字符,string表示的是字符串

多个单字符组合成一个序列,string就是从序列种生成的,
所以string有两种格式:

- 原始string
- 从序列中解释而得到的string

一个是无转义,一个是转义之后的字符串.\`abc\n\`  "abc\n"

""字符串,里面的字符都是解释过的rune字符

    string_lit             = raw_string_lit | interpreted_string_lit .
    raw_string_lit         = "`" { unicode_char | newline } "`" .
    interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
