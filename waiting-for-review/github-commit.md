# 高效编写git commit

为啥要特别说明commit，因为github的很多功能都可以在commit中用到。
特别是issues的操作，不过最重要的是基于issues的项目管理，提交日志非常重要。

一个好的git commit，配合github issues有以下优势：

- review 加速(知道为什么改/怎么改)
- release note的前身
- 某次提交修改了哪些功能，动了哪些代码，一目了然
- git blame 时提供足够信息
- 好的提交，提高项目的增题质量

## 基本要求

- 日志的第一行不要超过50个词
- 单独写提交日志，git commit -av就是不错的
- 日志中最好包含一个url，这个url指向项目的issue/stroy/card，这比issues号更好,
如果是基于github issues，那还是用单号比较号
- 日志中应该包含一个简短的用户故事，这样更容易被人理解

## 日志要表达的信息

- 为啥要改，改了哪些东西，让reviewers明白意图
- 如何改的，基本上是功能是如何实现的，如果大家都有共识就可省略
- 这次修改影响的范围，不要一次修改就修复10个bug，一次提交最多最多1-2个动作

## 最佳实践

- 使用fix/add/change 而不是fixed/added/changed
- 多行日志，第二行是空格
- 每次提交完成一个逻辑功能。如果可能，分解为多次小更新，每次更新易于理解是最好的。
- 基于github issues的提交，应该将单号放在头信息中(就是日志的第一行中)

## angular开源项目的日志规范

每次提交都应该包含以下3个部分：header/body/footer

    <type>(<scope>): <subject>   这个是header，是必须的，其他都是可选
    // 空一行
    <body>
    // 空一行
    <footer

- type 是必选，scope和subject都是可选的
- type 说明提交的类型，可以选择以下几种：
  - feat 新功能(feature)
  - fix 修复bug
  - docs 文档补充或文档变更
  - style 格式调整，不影响代码运行的变动
  - refactor 重构 (不是bug修复也不是新增功能)
  - test 增加测试项
  - chore 构建过程或辅助工具的变动
  - perf 提高性能
- 如果type是feat/fix，则改提交日志一定会出现在changelog中
- scope表明本次提交的影响范围，据具体项目而变，可以是数据层/业务层/视图层
- subject是提交的简短描述，不超过50个字符
  - subject 以动词开头，第一人称现在进行时，eg：change而不是changes/changed
  - 第一个字母大写
  - 结尾不加句号

body 是对提交日志的详细描述，可分层多行

    More detailed explanatory text, if necessary.  Wrap it to
    about 72 characters or so.

    Further paragraphs come after blank lines.

    - Bullet points are okay, too
    - Use a hanging indent

这这里会描述代码变动的动机/实现方式

footer只用于"不兼容变动"或关闭issue

    不兼容变动，一般指接口参数变化等"不向前兼容"部分
    用BREAKING CHANGE：表示
    BREAKING CHANGE: isolate scope bindings definition has changed.

    To migrate the code follow the example below:

    Before:

    scope: {
      myAttr: 'attribute',
    }

    After:

    scope: {
      myAttr: '@',
    }

    The removed `inject` wasn't generaly useful for directives
    so there should be no code using it.

    关闭单号
    Closes #123, #245, #992
