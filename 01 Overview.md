# `Julia`下高产的Web框架

https://genieframework.com/docs/genie/v5.11/tutorials/Overview.html

Genie是Julia语言的一个全栈web框架, 其开发目标是: 优秀的生产力, 高效的运行时性能, 以及默认的最佳实践和安全性.

Genie的架构和开发受到其它框架的最好特性的启发, 而不是参考其设计. Genie使用Julia的方式来做自己的事情: `Controllers`是普通的Julia模块, `Models`充分利用了类型和多分发. Genie的app也是Julia工程, 其版本和依赖性管理有关Julia的`Pkg`提供, 代码加载和重载通过`Revise`自动建立.

Genie同时受到Julia的"入手简单, 成长自由"哲学思想的启发, 通过允许开发者在`REPL`或notebook[^1]

上启动一个app, 或者简单地通过几行代码创建web服务和API.

随着项目变得越来越复杂, Genie允许渐近地添加更多结构, 这是通过微框架暴露诸如选路,日志, 视图模板等特性来实现的.

如果需要数据库持久化, Genie的ORM框架`SearchLight`可以随时添加. 最后, 可以使用完整的MVC结构来开发和维护更复杂的, 端到端的web应用.

**注意**: 由于notebook实际上是一个图形化REPL, 本文档中REPL能实现的notebook都能实现, 今后出现notebook的地方若无必要将不再声明.

[^1]:Jupyter Notebook, https://github.com/JuliaLang/IJulia.jl

