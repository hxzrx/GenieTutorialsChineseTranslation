# 读取POST荷载

https://genieframework.com/docs/genie/v5.11/tutorials/Working-with-POST-Payloads.html

Genie处理`POST`数据, **首先**要注册一个专门的路由来处理POST请求, **然后**, 当受到一个POST请求时, Genie会自动抽取出荷载, 并加入到`Router.params(:POST)`集合, 可以通过`Request.postpayload`方法来访问这个数据.

## 处理`form-data`荷载

```julia
using Genie, Genie.Renderer.Html, Genie.Requests

form = """
<form action="/" method="POST" enctype="multipart/form-data">
  <input type="text" name="name" value="" placeholder="What's your name?" />
  <input type="submit" value="Greet" />
</form>
"""

route("/") do
  html(form)
end

route("/", method = POST) do
  "Hello $(postpayload(:name, "Anon"))"
end

up()
```

`postpayload`文档, 与getpayload非常类似:

```
help?> postpayload
search: postpayload

  postpayload() :: Dict{Symbol,Any}

  A dict representing the POST variables payload of the request (corresponding to a form-data request)

  ──────────────────────────────────────────────────────────────────────────────────────────────

  postpayload(key::Symbol) :: Any

  Returns the value of the POST variables key.

  ───────────────────────────────────────────────────────────────────────────────────────────────

  postpayload(key::Symbol, default::Any)

  Returns the value of the POST variables key or the default value if key is not defined.
```

