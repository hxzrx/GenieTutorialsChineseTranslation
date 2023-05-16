# 使用Web Scoket工作

https://genieframework.com/docs/genie/v5.11/tutorials/Working-with-Web-Sockets.html

Genie为websocket上的C/S通信提供了强大的工作流. 系统隐藏了复杂的网络级通信, 暴露出类似于Genie MVC工作流的抽象: C/S在频道`channels`上交换消息(`channels`是`routes`的等价物).

## 注册频道`channels`

消息被映射到一个匹配的频道, 在其中被Genie的`Router`处理, 后者抽取出消息荷载并且调用指定好的句柄(控制器方法或函数). 对于大部分情况, `channels`是`routes`的等价物, 并且它们以类似的方式定义:

```julia
using Genie.Router
channel("/foo/bar") do
	# process request
end
channel("/baz/bax", YourController.your_handler)
```

这两个`channel`定义会处理发送到`/foo/bar`和`/baz/bax`的websocket消息.

## 设置客户端

为了在浏览器上启用WebSockets通信, 我们需要加载一个JavaScript文件, 这个文件通过`Assets`模块由Genie提供. Genie将客户端的WebScockets架构变得相当简单, 只需通过一个`Assets.channels_support()`方法. 例如, 假如要在web应用的主页上添加WebSockets支持, 需要做的而只是:

```julia
using Genie.Router, Genie.Assets
route("/") do
	Assets.channels_support()
end
```

## 实操

到Julia的REPL操作:

```julia
using Genie, Genie.Router, Genie.Assets

Genie.config.websockets_server = true # enable the websockets server

route("/") do
  Assets.channels_support()
end

up() # start the servers
```

访问http://127.0.0.1:8000/, 得到一个空白页面, 使用浏览器的开发工具(一般是快捷键Ctrl+Shift+i), 会发现页面加载了`channels.js`文件, 并且在控制台上打印了一条`Subscription ready`文本消息.

回到前面的代码, 调用`Assets.channels_support()`后, Genie会执行以下逻辑:

* 加载`channels.js`文件以提供WebScokets通信的JS API.
* 创建两个默认的频道, 用于订阅和取消订阅: `/_/subscribe`, `/_/unsubscrie`.
* 调用`/_/subscribe`并在C/S之间创建一个WebScokets连接.

注意Assets.channels_support()的返回: 

```julia
julia> Assets.channels_support()
"<script src=\"/genie.jl/master/assets/js/channels.js\"></script>"
```



## 从服务端推送消息

与客户端进行交互, 先要知道由哪些连接:

```julia
julia> Genie.WebChannels.connected_clients()

1-element Vector{Genie.WebChannels.ChannelClient}:
 Genie.WebChannels.ChannelClient(HTTP.WebSockets.WebSocket(UUID("dba75ab5-73f8-46d6-a265-80aadd12209c"), 🔁  519s 127.0.0.1:8000:8000 Base.Libc.WindowsRawSocket(0x0000000000000588), HTTP.Messages.Request:
......
```

会详尽地打印出每个连接的信息.

给其中的连接广播一条消息:

```julia
julia> Genie.WebChannels.broadcast("____", "Hey!")  # 第一个参数"____"要通过connected_clients()打印的信息查看, 位于信息的末尾
true
```

在浏览器的控制台上会看到打印了一条"Hey!"文本消息.

Genie.WebChannels.broadcast的帮助文档

```julia
help?> Genie.WebChannels.broadcast
  Pushes msg (and payload) to all the clients subscribed to the channels in channels, with the exception of except.
────────────────────────────────────────────────────────────────────────────────────
  Pushes msg (and payload) to all the clients subscribed to the channels in channels, with the exception of except.
```

由于可以从服务端重载window.parse_payload来处理消息, 继续在REPL上执行:

```julia
julia> route("/") do
           Assets.channels_support() *
           """
           <script>
           window.parse_payload=function(payload) {
               console.log('Got this payload: ' + payload);
           }
           </script>
           """
       end
```

刷新页面并重新发送消息`Genie.WebChannels.broadcast("____", "Hallo!")`, 控制台会按新的格式打印消息.

移除不可到达的客户端连接(例如关闭了浏览器这些失去连接的客户端):

```julia
julia> Genie.WebChannels.unsubscribe_disconnected_clients()

Dict{UInt64, Genie.WebChannels.ChannelClient} with 1 entry:
  0x0f77bb310de22930 => ChannelClient(WebSocket(UUID("60fc38b0-cf81-4ac8-bd05-3c228851d98a"), ......
```

**提示**: 应当例行调用unsubscribe_disconnected_clients()来释放内存.

## 从客户端推送消息

由于没有使用UI, 需要通过浏览器的控制台, 结合Genie的JS API来发送消息. 首先, 需要建立用于接收消息的频道`channel`. 只需在REPL上运行:

```julia
julia> channel("/____/echo") do
           "Received & send back from the server: $(params(:payload))"
       end
[WS] /____/echo => #11 | :_____echo
```

然后, 在浏览器的控制台上运行:

```js
Genie.WebChannels.sendMessageTo('____', 'echo', 'Hallo!')
```

会立即在浏览器的控制台上看到反馈消息:

```
Got this payload: Received & send back from the server: Hallo!
```

## 小结

本文为Genie使用WebSockets做了一个简单的引导, 现在应该有了足够的知识去建立C/S的通信, 互相发送消息, 以及使用`WebChannels`API去执行各种任务.





