# 加载和启动Genie应用

https://genieframework.com/docs/genie/v5.11/tutorials/Loading-Genie-Apps.html

Genie应用是Julia工程, 由分布于多个文件的多个模块组成. 加载Genie应用会带入所有文件的作用域, 包括主程序模块, 控制器模块, 模型模块, 等等.

由Genie.Generator生成的模板应用, 都可以从`bin/`下的文件启动.

这里假设控制台位于项目根目录.

## 在MacOS/Linux上启动Genie REPL会话

```shell
$ bin/repl   # 加载Genie应用
julia> up()  # 启动web服务器并进入REPL
```

如果想字节启动服务器而不进入REPL, 则在终端上执行:

```shell
$ bin/server  # 不进入REPL
```

## 在Windows上启动Genie REPL会话

windows上这是通过bat文件启动, 双击即可运行.

注意, 有时候在`bin/`目录下可能会缺少某个平台的启动文件, 这时需要重新生成:

```shell
julia> using Genie
julia> Genie.Generator.setup_windows_bin_files()  # win
julia> Genie.Generator.setup_nix_bin_files()      # *nix
julia> Genie.Generator.setup_windows_bin_files("path/to/your/Genie/project") # 指定文件路径
```

## 在REPL/Jupyter/Pluto/VSCode或其它Julia环境下启动

这时需要首先激活本地的包环境.

```julia
using Pkg;
Pkg.activate(".")
using Genie
Genie.loadapp()  # 加入已经位于app的根目录
Genie.loadapp("path/to/your/Genie/project") # 若app位于别处
up()
```

## 在REPL上手动加载

```shell
julia> ] # enter pkg> mode
pkg> activate .
julia> using Genie
julia> Genie.loadapp()
julia> up()
```

