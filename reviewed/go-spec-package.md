# package

一个go程序是由多个go包组成，
一个包是由多个源文件组成，
源文件里包含了常量/类型/变量/函数等，这些都属于同一个包，
同一个包的元素是可以相互访问的,
这些元素又可以导出给其他包用。

个人理解:最小颗粒度的隔离单元.

## 源文件的组织

每个源文件都有一个语句，证明自己归属哪个包，
之后就是import导入集合，这个集合表明这个包用到了哪些其他包，
之后是常量/变量/类型/函数的声明集。这些集都可能为空

    SourceFile = PackageClause ";" { ImportDecl ";" } { TopLevelDecl ";" } .

从ebnf中可以看出，源码文件主要包含3部分：包条款/导入声明/顶层声明

## 包条款

    PackageClause  = "package" PackageName .
    PackageName    = identifier .

ebnf中的标识符不能是空白标识符

多个源文件共享一个包名。

同一个包的源文件在同一目录

## 导入声明

只有先导入，后面才能使用.
也只能使用暴露的元素.

    ImportDecl       = "import" ( ImportSpec | "(" { ImportSpec ";" } ")" ) .
    ImportSpec       = [ "." | PackageName ] ImportPath .
    ImportPath       = string_lit .

ebnf中的PackageName,常用于qualified标识符(eg:fmt.Println,通过包名引用的标识符),
这样就可以访问包的暴露元素.如果省略了,那就通过包条款来引用暴露的元素.
当然PackageName也可以是".",表示可以不加前缀,直接引用暴露的元素,
不过最好不要这样.

还有一种不写包别名的情况，就是使用".",表示当前文件块可直接使用要导入的标识符，
不需要使用包名。这只是语法糖，最好不要这样使用。

导入路径就看具体实现，目前Go官方编译器支持很多远程url。
导入路径一般是编译路径的一个子串，也可能是一个相对路径。

    Import declaration          Local name of Sin

    import   "lib/math"         math.Sin
    import m "lib/math"         m.Sin
    import . "lib/math"         Sin
    // 3种不同的写法

导入声明，说的是一种依赖关系，不能出现导入自己，直接或间接的都不行，
导入了，又没有使用包中的标识符，也是不行的。

如果只是想要包的副作用(包的初始化)，可以使用空白标识符：

    import _ "lib/math"
