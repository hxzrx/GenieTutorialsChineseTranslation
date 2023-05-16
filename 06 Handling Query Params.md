# 处理查询参数(GET变量)

https://genieframework.com/docs/genie/v5.11/tutorials/Handling-Query-Params.html

### 简单举例

查询差是GET请求的URL上的一部分值, 例如`www.website.com/index?foo=1&bar=2`, 这里的foo和bat就是查询参数, 对应到Genie上的而变量`foo=1`, `bar=2`. 这些值会被Genie自动收集并且暴露在`Genie.Router.params()`集合上(Genie模块下也有params函数, 帮助文档都一样, 应该是相同的).

```
pkg> add Genie
julia> using Genie
julia> route("/hi") do
           name=params(:name, "Anon")  # params函数相当于CL的gethash, "Anon"是默认返回
           "Hello $name"
       end
julia> up()
```

通过http://127.0.0.1:8000/hi?name=Adrian访问, 这个请求的Adrian会作为name的参数传递给句柄函数. Genie中获取请求参数的函数是`Genie.params(:name)`.

# `Requests`模块

Genie在Requests模块中提供了一组处理请求数据的实用函数. 可以使用`getpayload`方法来检索查询参数, 查询参数是在Genie中表示成字典`Dict{Symbol, Any}`, 用Requests的实用函数实现上例功能:

```julia
julia> using Genie.Requests
julia> route("hallo") do
           "Hallo $(getpayload(:name, "Anna"))"
       end
```

其中, `getpayload`这个泛型函数有若干种特化方法, 上面用到的特化方法接受一个关键字和一个默认值, 默认值用于字典检索失败的返回.

```
help?> getpayload
search: getpayload

  getpayload() :: Dict{Symbol,Any}

  A dict representing the GET/query variables payload of the request (the part
  corresponding to ?foo=bar&baz=moo)

  ────────────────────────────────────────────────────────────────────────────

  getpayload(key::Symbol) :: Any

  The value of the GET/query variable key, as in ?key=value

  ────────────────────────────────────────────────────────────────────────────

  getpayload(key::Symbol, default::Any) :: Any

  The value of the GET/query variable key, as in ?key=value. If key is not
  defined, default is returned.
```



