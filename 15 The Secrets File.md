# 机密文件(`config/secrets.jl`)

https://genieframework.com/docs/genie/v5.11/tutorials/The-Secrets-File.html

机密配置数据(例如API密钥, 用户名, 密码, 等)应当添加到`config/secrets.jl`文件. 这个文件在创建Genie应用时默认是添加到`.gitignore`中的, 因此不会被提交上去.

## 作用域(Scope)

所有在`secrets.jl`文件中的定义(变量, 常量, 函数, 等)都会加载到App的模块中. 因此, 如果你的App名为`MyGenieApp`, 这些定义就会位于`MyGenieApp`名字空间.

**提示**: 假定App的名称是可变的, 仍然可以通过`Main.UserApp`常量来访问这个App的模块. 因此, 所有添加到`secrets.jl`文件中的定义, 也都可以通过`Main.UserApp`模块来访问.