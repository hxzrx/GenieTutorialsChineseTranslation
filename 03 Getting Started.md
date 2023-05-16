# Hello World

https://genieframework.com/docs/genie/v5.11/tutorials/Getting-Started.html

## 在REPL上交互式运行Genie

```julia
julia> route("/hallo") do
           "Hallo Welt!"
       end
julia> up()
```

**效果**: 打开浏览器访问地址http://127.0.0.1:8000/hallo, 将看到文本"Hallo Welt!", 注意访问主页http://127.0.0.1:8000返回404, 因为没有定义`"/"`路由.

**代码说明**: `route`函数定义了URL(这里是`"/hallo"`)到Julia句柄函数(`handler`)的映射, 因此访问对应地址看到"Hallo Welt!"文本.

**注意**: 由于Julia使用JIT编译, 函数再其第一次被调用时会自动编译, 因此第一次的相应会慢些, 但后面的都很快.

## 开发简单的Genie脚本

目标: 创建一个简单的"Hello World"微服务.

首先创建一个新目录, 在其中创建一个文件`geniews.jl`, 编辑如下代码:

```julia
using Genie, Genie.Renderer, Genie.Renderer.Html, Genie.Renderer.Json

route("/hello.html") do
    html("Hello World")  # 渲染函数html()位于Renderer.Html
end

route("/hello.json") do
    json("Hello World")  # 渲染函数json()位于Renderer.Json
end

route("/hello.txt") do
    respond("Hello World", :text)
end

up(8000, async = false)
```

运行: 控制台进入工程目录, 运行`julia geniews.jl`

代码前两个路由使用了渲染函数`html`, `json`.  渲染函数使用正确的格式和文档类型(使用正确的MIME[^1]), 负责数据的输出, 代码中hello.html使用HTML数据, hello.json使用JSON数据.

第三个路由负责文本相应, Genie对于`text/plain`相应没有提供专门的`text()`方法, 这里使用的是通用的`respond`函数, 并包含所期望的MIME类型, 即`:text`, 它对应于`text/plain`. 其它的MIME类型还有`:xml`, `:markdown`, `:javascript`等. 用户可以注册他自己想要的的mine类型和相应类型.

`up`函数在8000端口上启动web服务器. 这里, `async=false`表示服务器启动方式是同步的, 这会阻塞脚本的运行, 否则, 运行到脚本末尾, Julia进程就会正常退出, 服务器关闭.

注意这里还没到HMR, 对代码的任何修改都要重新运行脚本.

[^1]: MIME(Multipurpose Internet Mail Extensions)多用途互联网邮件扩展类型

## 小结

这里学到的是渲染函数和路由模块.

## 附: VS Code设置

1. 添加Julia语言支持: 在EXTENSIONS中安装Julia.
2. 在项目根目录下打开vscode, 在下方`TEMINAL`中运行julia, 按下`]`进入`pkg`.
3. 在`pkg`中运行`"activate ."`激活当前目录, 注意后面有一个点, 初次运行会在后台运行所加载包的索引, 运行完后编辑区域julia符号会有智能提示.
4. 退出`pkg`, 快捷键`Backspace`.
5. 格式化代码, 通过快捷键`Alt+Shift+f`.



























