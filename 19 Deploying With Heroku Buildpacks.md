# 通过Heroku Buildpacks布署Genie应用

https://genieframework.com/docs/genie/v5.11/tutorials/Deploying-With-Heroku-Buildpacks.html

## 预备条件

要有一个Heroku账号并注册了[Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).

## 应用

要布署应用, 首先要有一个样例应用, 要么使用自己的应用, 要么克隆下面得样例.

### 所有的步骤(只需Ctrl+c / Ctrl+v)

1. 取一个惟一得名称: `HEROKU_APP_NAME=my-app-name`
2. 作为演示, 克隆样例应用: `git clone https://github.com/milesfrain/GenieOnHeroku.git`
3. 进入应用根目录: `cd GenieOnHeroku`
4. 创建一个Heroku应用: `heroku create $HEROKU_APP_NAME --buildpack https://github.com/Optomatica/heroku-buildpack-julia.git`
5. 把新建得应用放到Heroku: git push heroku master
6. 至此, 可以在浏览器上打开应用: heroku open -a $HEROKU_APP_NAME
7. 查看日志: heroku logs -tail -a $HEROKU_APP_NAME

## 更具体的步骤描述

### 取名

```
HEROKU_APP_NAME=my-app-name
```

名称必须在Heroku的所有工程上惟一, 因为它是项目托管url的一部分(例如: `https://my-app-name.herokuapp.com/`). 如果名称不惟一, 在`heroku create`步会报错.

### 克隆项目

```shell
git clone https://github.com/milesfrain/GenieOnHeroku.git
cd GenieOnHeroku
```

项目根目录上必须含有加载应用的`Procfile`文件, 文件内容为:

```shell
web: julia --project src/app.jl $PORT
```

也可以使用自己的加载脚本, 要考虑到动态改变的`$PORT`环境变量, 它由Heroku设置.

如果是发布由`Genie.newapp`生成的Genie应用, 其启动脚本是`bin/server`, Genie会自动从环境中获得`$PORT`端口号.

### 创建Heroku工程

```shell
heroku create $HEROKU_APP_NAME --buildpack https://github.com/Optomatica/heroku-buildpack-julia.git
```

这会在Heroku平台创建一个工程, 其中包括一个单独的git仓库.

同时, heroku仓库也在仓库列表中, 可通过`git remote -v`查看.

为Julia使用buildpack, 这会运行Julia工程需要的常用的部署操作, 依赖于工程根目录下的`Project.toml`, `Manifest.toml`, 以及`src`目录下的Julia代码.

### 布署应用

```shell
git push heroku master
```

这会将当前本地分支推送到`heroku`的远程`master`分支.

Heroku会中自动执行描述在Julia的Julia的buildpack和Procfile中的命令.

自动布署的触发必须是推送到heroku的`master`分支.

### 打开App的网页

```shell
heroku open -a $HEROKU_APP_NAME
```

这是打开浏览器的一个快捷命令.

App的页面地址是: `https://$HEROKU_APP_NAME.herokuapp.com/`

### 查看App的日志

```shell
heroku logs -tail -a $HEROKU_APP_NAME
```

这是启动日志观察器的一个快捷命令, Julia的`println`日志也会包含在这.

从Heroku的网页监控面板上也可以查看日志, 例如: `https://dashboard.heroku.com/apps/my-app-name/logs`

App的更新布署

布署对App的修改, 只需简单地在本地做修改, 然后通过git推送给heriku.

```shell
git push heroku master
```

