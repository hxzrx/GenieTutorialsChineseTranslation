# 高级路由技术

https://genieframework.com/docs/genie/v5.11/tutorials/Advanced-Routing-Techniques.html

**路由(router)**是Genie的核心, 它负责web请求与句柄函数的匹配, 负责请求变量的抽取和设置, 负责环境的执行, 以及响应方法的调用.

## 静态路由

从最简单的例子入手, 可以使用`route`方法注册"明文(plain)"路由. `route`方法第一个参数是路径, 第二个参数是句柄函数.

```julia
using Genie
greet() = "Willkommen zum Genie"
route("/greet", greet)  # http://127.0.0.1:8000/greet
up()  # 起服
```

更简单的, 可以这么实现:

```julia
route("/bye") do  # http://127.0.0.1:8000/bye
    "Tschüss!"
end
```

注意: 

1. 路由的匹配是从最新到最老的方向, 因此可以定义新的路由把老的路由替换掉.
2. Genie的路由匹配规则不是匹配最特定的, 而是使用第一个匹配, 因此, 如果注册了对`/*`的路由, 它能够匹配所有的请求, 即使前面有更特定的路由, 这一定要注意!! 这个技术一般用于站点的维护(把所有页面都定向到同一个页面).
3. 路由的删除: `Router.delete!(:route_name)`.

```julia
julia> route("/*") do  # 访问http://127.0.0.1:8000/bye
           "Was?"
       end
julia> routes()  # 查看所有路由
julia> Router.delete!(Symbol("get_*"))  # 删除'/*'路由
```

## 动态路由(使用路由参数)

静态路由适合于固定URL, 但对于动态URL, 每个请求都不固定, 因此得用动态路由.

这些情形下要通过动态路由或者路由参数来处理, 例如, `"/customers/57943/orders/458230"`, 可以定义动态路由为`"/customers/:customer_id/orders/:order_id"`, 在匹配请求时, 路由会解出这些值并放到`params`集中.

```julia
julia> using Genie.Requests
julia> route("/customers/:customer_id/orders/:order_id") do
           "U asked for the order $(payload(:order_id)) for customer $(payload(:customer_id))"
       end
```

`payload`文档

```julia
help?> payload
search: payload ispayload rawpayload getpayload jsonpayload filespayload infilespayload postpayload
  payload() :: Any
  Utility function for accessing the params collection, which holds the request variables.
  ────────────────────────────────────────────────────────────────────────────────────────────────
  payload(key::Symbol) :: Any
  Utility function for accessing the key value within the params collection of request variables.
  ──────────────────────────────────────────────────────────────────────────────────────────────────
  payload(key::Symbol, default_value::T) :: Any
  Utility function for accessing the key value within the params collection of request variables. If key is not defined, default_value is returned.
```

# 路由方法(GET, POST, PUT, PATCH, DELETE, OPTIONS)

默认下, 路由处理的是GET请求, 因为这是最常见的. 为了定理其他类型的请求, 定义路由是传递一个`method`关键字参数, 指定想要响应的HTTP方法. Genie的路由支持GET, POST, PUT, PATCH, DELETE, OPTIONS方法. 这几个常量都是在`Router`模块下, 并且被导出.

### 举例说明

```julia
julia> using Genie, Genie.Requests
julia> route("/patch_stuff", method=PATCH) do # 注意PATCH是一个字符串
           "stuff to patch"
       end
julia> up()
julia> HTTP.request("PATCH", "http://127.0.0.1:8000/patch_stuff").body |> String  # PATCH不能是小写
[ Info: PATCH /patch_stuff 200
"stuff to patch"
```

代码输出字符串"stuff to patch"作为`PATCH`请求的响应.

## 具名路由(Named routes)

首要要明确, 路由的名称是符号类型!

Genie允许给路由赋予一个名称, 这是个非常强大的特性, 它与`Router.tolink`方法一起使用, 可以为动态创建URL匹配到不同的路由. 这项技术的优势在于, 如果我们偏好于通过名称和用`tolink`动态生成链接的路由, 只要路由的名称保持不变, 如果改变了路由的模式, 所有的URL都会自动到匹配新的路由.

```julia
julia> route("/customers/:customer_id/orders/:order_id", named=:get_customer_order) do
           "Looking uo order $(payload(:order_id)) for customer $(payload(:customer_id))"
       end
[GET] /customers/:customer_id/orders/:order_id => #5 | :get_customer_order
julia> routes()
2-element Vector{Genie.Router.Route}:
 [GET] /customers/:customer_id/orders/:order_id => #5 | :get_customer_order
 [PATCH] /patch_stuff => #3 | :patch_patch_stuff
```

注意: 为了保持一致性, Genie给每个路由都赋予了名称, 但是自动生成的名称是与状态相关的, 因此如果改变了路由的定义, 它的名称也可能会随之改变, 因此最好就显示地指定路由地名称.

```julia
julia> route("/foo") do  # 自动生成名称:get_foo
         "foo"
       end
julia> routes()
3-element Vector{Genie.Router.Route}:
 [GET] /foo => #7 | :get_foo
 [GET] /customers/:customer_id/orders/:order_id => #5 | :get_customer_order
 [PATCH] /patch_stuff => #3 | :patch_patch_stuff
```

## 链接到路由(Links to routes)

使用**`linkto`**方法可以把路由地名称链接回路由本身. 因此, 首先需要知道路由的名称, 由于名称是个符号, 就需要能够表示这个符号, 一般可以通过符号字面量的语法糖访问, 但有些特殊的符号不能这样, 就得通过Symbol("name_string")来获得这个符号, 例如`route("/*")`会自动生成名称`:get_*`, 但由于存在星号而不能用这个字面量来表示, 因此必须通过Symbol("get_*")来获得这个符号.

```julia
julia> routes()
3-element Vector{Genie.Router.Route}:
 [GET] /foo => #7 | :get_foo
 [GET] /customers/:customer_id/orders/:order_id => #5 | :get_customer_order
 [PATCH] /patch_stuff => #3 | :patch_patch_stuff
julia> Router.linkto(:patch_patch_stuff)
"/patch_stuff"
julia> linkto(:get_customer_order, customer_id=1234, order_id=5678)
"/customers/1234/orders/5678"
```

`linkto`函数返回的是一个字符串, 他也可以用于HTML模板中用于生成链接, 例如:

```html
<a href="$(linkto(:get_foo))">Foo</a>
```

## 列举出路由

使用`Route.routes`函数.

```julia
julia> routes()
3-element Vector{Genie.Router.Route}:
 [GET] /foo => #7 | :get_foo
 [GET] /customers/:customer_id/orders/:order_id => #5 | :get_customer_order
 [PATCH] /patch_stuff => #3 | :patch_patch_stuff
```

### `Route`类型

在Genie中, 路由通过`Route`类型来表示, 它具有以下字段:

- method::String - 存储路由的方法(GET, POST, 等六种)
- path::String - 表示用于匹配的URI模式
- action::Function - 但路由得到匹配时调用的句柄函数
- name::Union{Symbol, Nothing} - 路由的名称
- context::Module - 一个可选的上下文, 在调用句柄函数时要使用

## 移除路由

通过调用`delete!`方法来移除指定参数名称的路由, 方法返回删除后的路由集.

```julia
julia> routes()
3-element Vector{Genie.Router.Route}:
 [GET] /foo => #7 | :get_foo
 [GET] /customers/:customer_id/orders/:order_id => #5 | :get_customer_order
 [PATCH] /patch_stuff => #3 | :patch_patch_stuff
```

文档

```julia
help?> Router.delete!
  delete!(route_name::Symbol)
  Removes the route with the corresponding name from the routes collection and returns the collection of remaining routes.
```

## 通过参数类型匹配路由

默认下, 路由的参数都是以`Substring{String}`类型解析到`payload`集. 下面代码可以看出:

```julia
using Genie, Genie.Requests
route("/customers/:customer_id/orders/:order_id") do
  "Order ID has type $(payload(:order_id) |> typeof) // Customer ID has type $(payload(:customer_id) |> typeof)"
end
```

访问http://127.0.0.1:8000/customers/Anna/orders/20, 得到`Order ID has type SubString{String} // Customer ID has type SubString{String}`.

但是在有些情况下(例如上例), 接收`Int`数据会更好, 则可以避免显示的类型转换, 并且只需要匹配整数类型. Genie指出这种工作流, 只需通过路由参数的类型标注即可实现:

```julia
route("/customers/:customer_id::Int/orders/:order_id::Int", named = :get_customer_order) do
  "Order ID has type $(payload(:order_id) |> typeof) // Customer ID has type $(payload(:customer_id) |> typeof)"
end
```

注意参数标注成`Int`. 访问http://127.0.0.1:8000/customers/001/orders/20, 得到`Order ID has type Int64 // Customer ID has type Int64`.(英文文档中的报错并未发现)

### 选路函数中的类型转换

本小节是整对上例在英文文档中报错的解决方案, 但最新版的Genie已能够正确进行类型转换. 但这里的方法可为一般性的类型转换提供方法:

```julia
Base.convert(::Type{Int}, v::SubString{String}) = parse(Int, v)
```

代码把数据从类型`SubString{String}`转换成`Int`.

## 匹配单独的URI片段

除了能够匹配完整的路由, Genie还允许匹配单独的URI片段. 即, 迫使各个路由参数遵循某个模式. 为了给路由参数引入约束, 需要在路由参数末尾添加一个模式`#pattern`.

### 举例说明

假设我们想要实现一个具有本地语言的站点, 其中的URL结构类似于: `mywebsite.com/cn`, `mywebsite.com/en`, `mywebsite.com/de`. 可以定义一个动态路由函数, 抽取"地区"变量来提供本地化内容的服务.

```julia
route(":locale#(en|es|de)", TranslationsController.index)
```

这个路由函数只允许`:locate`匹配`en`, `es`, `de`字符串.

也可以做成配置, 使得增加新语言后不需修改路由函数.

```julia
const LOCALE = ":locale#($join(TranslationsController.AVAILABLE_LOCALES, '|')))"
route("/$LOCALE", TranslationController.index, named=:get_index)
```

## `params`集

路由函数将请求中的所有参数都打包进一个`Dict{Symbol, Any}`类型的`params`集, 其中包含有价值的信息, 例如路由参数, 查询参数, POST荷载, 原始的`HTTP.Request`和`HTTP.Response`对象, 等等.

一般来说, 不建议直接访问`params`集, 而是通过定义在`Genie.Requests`和`Genie.Responses`上的实用方法. 但对高级用户来说, 了解`params`集有时也会带来方便.
