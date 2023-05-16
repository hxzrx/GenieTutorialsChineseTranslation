# 自己的Julia代码添加到`lib/`目录

https://genieframework.com/docs/genie/v5.11/tutorials/Publishing-Your-Julia-Code-Online-With-Genie-Apps.html

如果有自己的Julia代码要与Genie应用集成, 最简单的就是把文件添加到`lib/`目录. `lib/`目录及其子目录下的文件或被自动加载. 但要注意的是, App启动后再添加的文件和目录, 需要重启App才能生效.

**提示**: Genie不会自动生成`lib/`目录, 需要手动创建.

```julia
# lib/MyLib.jl
module MyLib

using Dates

function isitfriday()
  Dates.dayofweek(Dates.now()) == Dates.Friday
end

end
```

添加后, 这个新的`MyLib`模块会被放置在顶层模块()下, 假设顶层模块为`MyGenieApp`, 则新的模块通过`MyGenieApp.MyLib`访问. 于是, 新模块的使用就可以这样:

```julia
# routes.jl
using Genie
using MyGenieApp.MyLib # or using ..Main.UserApp.MyLib

route("/friday") do
  MyLib.isitfriday() ? "Yes, it's Friday!" : "No, not yet :("
end
```

