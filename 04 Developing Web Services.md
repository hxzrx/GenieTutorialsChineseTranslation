# 用Genie开发Web服务

https://genieframework.com/docs/genie/v5.11/tutorials/Developing-Web-Services.html

在REPL上随手启动一个web服务并编写小量的脚本很方便, 但不实用.

Genie允许通过模块化的方法进行应用的构建, 允许在需要时才去添加更多组件. 可以开始于一个web服务模板(包含依赖性管理, 日志, 环境, 路由), 并按顺序添加DB持久化(通过Genie的ORM层SearchLight), 添加高性能HTML视图模板(通过Rdnderer.HTML), 添加异常捕捉, 添加验证, 等等.

##  建立Web服务项目

首先初始化一个Genie项目. 将控制台进入到一个空目录上, 运行Julia.

```julia
julia> using Genie
julia> Genie.Generator.newapp_webservice("MyGenieApp")
```

这会创建一个`MyGenieApp`的目录, 并在其中生成一堆文件, 然后以[默认地址和端口](http://127.0.0.1:8000)启动web服务器. 其中有几个重要的文件:

1. `Project.toml`: Julia的包定义文件, 包含显示依赖的包定义. 这文件类似于NPM项目的package.json文件.
2. `Manifest.toml`: 有Julia自动生成, 包含隐私依赖的包定义. 这两个文件用于管理项目的文件依赖.

**目录结构**:

1. `bin/`目录包含启动Genie REPL或Genie服务器的shell脚本和bat文件.
2. `bootstrap.jl`文件以及`src/`目录用于加载应用.
3. `config/`目录包含环境配置, 即运行环境(dev, prod, test, global).
4. `public/`目录是文档的根目录, 包含web服务的静态文件.
5. `routes.js`文件负责Genie路由的注册.
6. `test/`目录放置测试用例.

**提示**: Julia的帮助文档, 通过快捷键`Shift+/`进入, 会看到提示符`help?>`, 在其中键入`Genie.Generator.newapp_webservice`会得到对应的文档. 除`newapp_webservice`之外, `Genie.Generator`下还有其它的生成函数, 后面会讲.

## 添加代码逻辑

向`routes.jl`添加一个路由逻辑.

```julia
route("/hello") do
  "Freut mich!"
end
```

保存后访问http://127.0.0.1:8000/hello立即看到效果, 有了HMR功能.

## 扩展APP

Genie的App也是Julia的项目, 因此可以通过Julia的方式对其进行扩展, 例如在``pkg`中添加新的依赖库.

如果有现成的Julia想要添加到Genie的App中, 可以在项目根目录下创建一个`lib/`目录并把代码放到这里. `lib/`下的文件会被Genie自动加载, 不需要考虑文件深度. 

注意: 在运行中添加`lib/`目录要重启服务器后才会加载其中的文件.

### 添加数据库支持

在REPL中运行

```julia
julia> Genie.Generator.db_support() # 这条命令的执行会比较久
```

**注意**: 如果准备创建一个复杂的项目, 这样一点一点添加功能会很麻烦, 建议一开始就使用Genie的面向资源的MVC结构会更省去一些杂活.