# 配置

在某个包的目录下创建一个文本文件 xx.goconvey

这个文件里可以包含：

- 空行
- 注释行，用#或//开头即可
- test flag 就是go test的flag
- ignore (ignore必须在test flag前面)

如果文件内容(除开空行和注释行)第一行是ignore，那这个包就会被忽略。
其他情况，文件内容都会被当成go test 的flag

下面是一些通用惯例：

- 就算文件里没有-v，默认都会添加上
- -cover/-covermode/-coverprofile 这几个参数由goconvey服务指定，文件指定无效
- -tags会传给go test，需要保证正确
