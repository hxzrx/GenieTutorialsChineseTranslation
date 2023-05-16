# 使用JSON荷载

https://genieframework.com/docs/genie/v5.11/tutorials/Using-JSON-Payloads.html

通过`POST`请求将`JSON`荷载以`application/json`数据发送, 是一种非常常见的设计模式, 尤其是在开发REST API中. Genie通过实用函数`Requests.hsonpayload`处理这种情况. 在背后, Genie处理`POST`请求并尝试去解析JSON文本荷载(解析成字典`Dict`类型), 如果失败了(指文本荷载无法转换成JSON), 仍然可以通过`Requests.rawpayload`访问原始数据.

## 举例说明

```julia
using Genie, Genie.Requests, Genie.Renderer.Json

route("/jsonpayload", method = POST) do
  @show jsonpayload()
  @show rawpayload()

  json("Hello $(jsonpayload()["name"])")
end

up()
```

然后创建一个POST请求:

```julia
pkg> add HTTP
julia> using HTTP
julia> HTTP.request("POST", "http://127.0.0.1:8000/jsonpayload", [("Content-Type", "application/json")], """{"name": "Anna"}""")

jsonpayload() = Dict{String, Any}("name" => "Anna")
rawpayload() = "{\"name\": \"Anna\"}"
[ Info: POST /jsonpayload 200
HTTP.Messages.Response:
"""
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Server: Genie/Julia/1.9.0
Transfer-Encoding: chunked

"Hello Anna""""
```

`@show`文档

```
help?> @show
  @show exs...

  Prints one or more expressions, and their results, to stdout, and returns
  the last result.

  See also: show, @info, println.

  Examples
  ≡≡≡≡≡≡≡≡≡≡

  julia> x = @show 1+2
  1 + 2 = 3
  3

  julia> @show x^2 x/2;
  x ^ 2 = 9
  x / 2 = 1.5
```

