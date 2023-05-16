# `lib/`目录

https://genieframework.com/docs/genie/v5.11/tutorials/The-Lib-Folder.html

Genie能够很容易地将标准Genie MVC架构之外的Julia代码(模块, 文件, 等)自动加载进App, 只需简单地将文件和目录放到`lib/`目录下.

**提示**

* 如果`lib/`目录不存在, 就需要手动创建`julia> mkdir("lib")`
* Genie将`lib/`目录及其子目录下文件递归地进行加载
* `lib/`下的文件如果被修改了会通过`Revise`自动加载