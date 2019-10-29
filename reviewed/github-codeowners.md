# github的code owners

这个功能的灵感来至于google。  
CODEOWNERS定义了负责仓库代码的个人或团队

- 位置：docs/ 或 .github/
- 文件名：CODEOWNERS
- CODEOWNERS 语法和.gitignore[语法](https://git-scm.com/docs/gitignore#_pattern_format)类似
- 不同的分支可以有不同的CODEOWNERS

```.gitignore
# This is a comment.
# Each line is a file pattern followed by one or more owners.

# These owners will be the default owners for everything in
# the repo. Unless a later match takes precedence,
# @global-owner1 and @global-owner2 will be requested for
# review when someone opens a pull request.
*       @global-owner1 @global-owner2

# Order is important; the last matching pattern takes the most
# precedence. When someone opens a pull request that only
# modifies JS files, only @js-owner and not the global
# owner(s) will be requested for a review.
*.js    @js-owner   涉及js修改的，通知xx

# You can also use email addresses if you prefer. They'll be
# used to look up users just like we do for commit author
# emails.
*.go docs@example.com  涉及go修改的，通知xx，用户名和邮箱都是支持的

# In this example, @doctocat owns any files in the build/logs
# directory at the root of the repository and any of its
# subdirectories.
/build/logs/ @doctocat  可以只指定某个目录的所有者

# The `docs/*` pattern will match files like
# `docs/getting-started.md` but not further nested files like
# `docs/build-app/troubleshooting.md`.
docs/*  docs@example.com     docs下的文件指定所有者，但不嵌套

# In this example, @octocat owns any file in an apps directory
# anywhere in your repository.
apps/ @octocat   apps下的所有文件都被所有者所访问,相对路径，会做匹配

# In this example, @doctocat owns any file in the `/docs`
# directory in the root of your repository.
/docs/ @doctocat   /docs下的所有文件，此处是绝对路径
```
