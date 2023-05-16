# 外部包的管理

https://genieframework.com/docs/genie/v5.11/tutorials/Managing-External-Packages.html

Genie通过Julia的包管理器`Pkg`对外部包进行管理.

## 添加新包

步骤:

1. 启动Genie的REPL, `$bin/repl`, 则会自动加载App的包环境.
2. 切换到`Pkg`模式: `julia> ]`
3. 添加依赖包, `pkg> add NewPackage`

至此, 可以在REPL 上使用此新包`using NewPackage`, 或者在文件中引入`import NewPackage`.

## 更新已有包

```shell
pkg> up              # 更新所有包
pkg> up SomePackage  # 更新某特定的包
```

