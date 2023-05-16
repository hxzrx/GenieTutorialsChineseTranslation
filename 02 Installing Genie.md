# 如何安装Genie

https://genieframework.com/docs/genie/v5.11/tutorials/Installing-Genie.html

从Julia包管理器安装

```julia
pkg> add Genie
```

注: 从控制台启动julia后, 按`]`键进入`pkg`包管理, 安装完毕后按`Backspace`键回到`REPL`.

Genie的版本号为`vX.Y.Z`, 其意义为:

- X: 主版本, 包含不兼容改动.
- Y: 次版本, 带入新特性, 但不会破坏兼容性.
- Z: 补丁版本, 修改bug, 但不会带人新特性, 更不会破坏兼容性.

翻译时, Genie文档对应的Genie版本号是`v5.11`.