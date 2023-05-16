# 通过Docker实用Genie

https://genieframework.com/docs/genie/v5.11/tutorials/Using-Genie-With-Docker.html

Genie对Docker容器的支持是通过官方的`GenieDeployDocker`插件实现的.

## 设置`GenieDeployDocker`

为实用Docker集成特性, 首先要给Genie添加`GenieDeployDocker`插件.

```julia
pkg> add GenieDeployDocker
```

## 生成Genie优化的`Dockerfile`

可以通过调用`GenieDeployDocker.dockerfile()`函数来引导Docker的安装. 这会为Genie的web应用的容器化生成一个优化了的定制版`Dockerfile`文件. 这个文件会在当前工作目录下(或者通过`path`参数来指定, 参考`dockerfile()`函数).

一旦生成了这个文件, 就可以按需要进行编辑--Genie不会覆盖这个文件, 任何手动修改都会保留在`Dockerfile`文件上, 除非再次调用`dockerfile`并传递`force=true`参数). 

*注*: 这是这目录中已有`Dockerfile`文件的情况下, 再次调用`GenieDeployDocker.dockerfile()`函数不会对原文件写入任何东西(在REPL上第二次运行该函数会报错), 但调用``GenieDeployDocker.dockerfile(force=true)`会重新生成一个`Dockerfile`文件并把原来的覆盖掉.

`dockerfile()`函数的行为可以通过传递进去的参数来控制. 其文档:

```julia
help?> GenieDeployDocker.dockerfile
  dockerfile(path::String = "."; user::String = "genie", env::String = "dev",
            filename::String = "Dockerfile", port::Int = 8000, dockerport::Int = 80, force::Bool = false)
  Generates a Dockerfile optimised for containerizing Genie apps.
  Arguments
  ≡≡≡≡≡≡≡≡≡≡≡
    •  path::String: where to generate the file
    •  filename::String: the name of the file (default Dockerfile)
    •  user::String: the name of the system user under which the Genie app is run
    •  env::String: the environment in which the Genie app will run
    •  host::String: the local IP of the Genie app inside the container
    •  port::Int: the port of the Genie app inside the container
    •  dockerport::Int: the port to use on the host (used by the EXPOSE directive)
    •  force::Bool: if the file already exists, when force is true, it will be overwritten
```

## 构建Docker容器

一旦准备好了`Dockerfile`文件, 就可以调用`GenieDeployDocker.build()`来构建Docker容器. 可以传递任何受支持的可选参数来配置设置, 例如容器名称(默认名称是"genie"), 例如路径(默认是当前工作目录), 以及其它的参数, 需要时可以寻求帮助文档`help?>`.

## 在Docker容器上运行Genie应用

但镜像准备好了后, 就可以通过`GenieDeployDocker.run()`来运行. 这个函数也有多个可选参数用于控制app的运行方式, 具体的可以参考帮助文档`help?>`.

## 举例说明

首先创建一个Genie应用:

```julia
pkg> add Genie
julia> using Genie
julia> Genie.Generator.newapp("DockerTest")  # 创建App
pkg> add GenieDeployDocker
julia> using GenieDeployDocker
julia> GenieDeployDocker.dockerfile()  # 生成Dockerfile文件
julia> GenieDeployDocker.build()       # 构建容器
julia> GenieDeployDocker.run()         # 在容器中运行app
```

注意, 必须先安装Docker, windows版Docker[安装须知](https://docs.docker.com/desktop/install/windows-install/).

## 容器的审查

使用`GenieDeployDocker.list()`可获得一组可用的容器, 默认下只是显示当前正在运行的容器, 通过传递`all=true`参数能够看到未运行的容器.

```julia
julia> GenieDeployDocker.list()
```

## 停止运行容器

使用`GenieDeployDocker.stop()`函数, 传递容器名称进去, 可以停止正在运行的容器:

```julia
julia> GenieDeployDocker.stop("DockerTest")
```

## 在开发中使用容器

如果想要在开发中使用容器, 就要从主机中将这个app载入(mount)到容器中, 则会使得可以在本地持续地编辑文件, 同时对文件所作的改动也会反映到Docker容器中. 为达到这个目的, 只需将`mountapp=true`参数传递给`GenieDeployDocker.run()`:

```julia
julia> GenieDeployDocker.run(mountapp = true)
```

以这种方式启动后, 就可以对文件进行编辑, 所作的改动都会反映到Docker容器上.

## 通过`PackageCompiler.jl`创建优化版Genie系统镜像

则需要修改`Dockerfile`文件, 按照以下步骤:

### i. 编辑`Dockerfile`文件

1. 在`WORKDIR /home/genie/app`行之下, 添加

   ```
   # C compiler for PackageCompiler
   RUN apt-get update && apt-get install -y g++
   ```

2. 在`RUN julia -d`行之下, 添加

   ```
   # Compile app
   RUN julia --project compiled/make.jl
   ```

3. 针对部署环境, 修改运行环境

   ```
   ENV GENIE_ENV "prod"
   ```

### ii. 添加`PackageCompiler.jl`

添加`PackageCompiler`依赖:

```julia
pkg> add PackageCompiler
```

### iii. 添加必需文件

创建目录并添加文件:

```
julia> mkdir("compiled")
julia> touch("compiled/make.jl")
julia> touch("compiled/packages.jl")
```

### iv. 编辑文件

#### `packages.jl`文件:

```julia
# packages.jl
const PACKAGES = [
  "Dates",
  "Genie",
  "Inflector",
  "Logging"
]
```

#### `make.jl`文件

```julia
# make.jl
using PackageCompiler
include("packages.jl")

PackageCompiler.create_sysimage(
  PACKAGES,
  sysimage_path = "compiled/sysimg.so",
  cpu_target = PackageCompiler.default_app_cpu_target()
)
```

#### 使用预先编译好的镜像

上述文件编辑的结果是, `PackageCompiler`会创建一个新的Julia子镜像, 它会存放在`compiled/sysimg.so`文件中. 最后一步是告知`bin/server`脚本使用这个镜像.

编辑`bin/server`文件, 类似于:

```shell
julia --color=yes --depwarn=no --project=@. --sysimage=compiled/sysimg.so -q -i -- $(dirname $0)/../bootstrap.jl -s=true "$@"
```

注意其中新加的`--sysimage`参数, 表明要使用新的Julia子镜像.



