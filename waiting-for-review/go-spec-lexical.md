# 词法的组成元素

## 注释

注释作为程序的文档,有两种写法

    行注释 //
    块注释 /*  */

块注释和空格等价,行注释和新行等价

## tokens

令牌? 标记

tokens组成了go语言的词汇,tokens包括:

- 4各类:标识/关键字/操作符/标点
- 文字

空白由下面几个部分组成

- 空格
- tab
- 回车
- 换行

空白作为单独的tokne出现,一般会被忽略,可和其他token组合成新的token

换行/EOF,可能会出发;的插入

## 分号

;的出现,意味着一个productions的结束,productions是ebnf的一个概念

go会在下面两种情况下,忽略;的输入:

- 输入被分解为token,多个token会组成一条go语句,
这个go语句被称为(token流),如果再token流中,
找到了一个final token,会自动添加一个分号;
final token可以是下面几个:
  - 标识
  - int/float/复数/rune/string 文字
  - 关键字:break/continue/fallthrough/return
  - ++ -- ) ] }
- 复杂语句占一行,在 ) } 前面,可以忽略掉分号

## 标识 identifiers

- identifiers是一个程序的实体,eg:变量/类型
- identifiers由一个或多个字母数字组成的序列,第一个字符需要是字母

ebnf的定义是:

    identifier = letter { letter | unicode_digit } .

identifiers可以由下划线开头, 前面也说了,下划线属于字母

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

## 整数数字字符

    int_lit     = decimal_lit | octal_lit | hex_lit .
    decimal_lit = ( "1" … "9" ) { decimal_digit } .
    octal_lit   = "0" { octal_digit } .
    hex_lit     = "0" ( "x" | "X" ) hex_digit { hex_digit } .

数字,前可以添加前缀表示进制

## 浮点字符

    float_lit = decimals "." [ decimals ] [ exponent ] |
                decimals exponent |
                "." decimals [ exponent ] .
    decimals  = decimal_digit { decimal_digit } .
    exponent  = ( "e" | "E" ) [ "+" | "-" ] decimals .

## 复数字符

    imaginary_lit = (decimals | float_lit) "i" .

## rune字符

rune就是单引号包围的一个或多个字符,eg: '\n' 'x',
在单引号内,不能出现两种字符:换行和未转义的单引号.
一个单引号包裹的字符,就是字符的unicode值,和 \多个字符 的写法是同一个东西.

单引号包裹的单字符,是最常见的写法,go源码是unicode字符集,而且是utf-8编码,
多个utf-8不编码的字节,才能表示一个整数.

如果值阿ascii文档中,表示一个整数有4种写法:

- \x 后跟2个十六进制数 eg: \x3f
- \u 后跟4个十六进制数 eg: \u3f2c
- \U 后跟4个十六进制数 eg: \U2d1a
- \  后跟3个八进制数 eg: \632

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

所以rune是一种字符的表示方式,不仅仅包含了unicode写法,还包含其他写法

## string 字符

就是字符串,rune表示的单个字符,string表示的是字符串

多个单字符组合成一个序列,string就是从序列种生成的,
所以string有两种格式:

- 原始string
- 从序列中解释而得到的string

一个是无转义,一个是转义之后的字符串.\`abc\n\`  "abc\n"

""字符串,里面的字符都是解释过的rune字符

    string_lit             = raw_string_lit | interpreted_string_lit .
    raw_string_lit         = "`" { unicode_char | newline } "`" .
    interpreted_string_lit = `"` { unicode_value | byte_value } `"` .
