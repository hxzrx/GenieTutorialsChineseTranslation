# 通过Nginx把Genie应用布署到服务器

https://genieframework.com/docs/genie/v5.11/tutorials/Deploying-Genie-Server-Apps-with-Nginx.html

本篇展示如何通过Nginx托管Julia/Genie.

## 先决条件

为了将App暴露到因特网, 首先要能访问到服务器, 这可能是一台本地的机器, 或是一个远程的云实例.

如果是使用一个本地服务器, 就需要有一个静态IP来保证对App的持续性访问.

## 应用

这里假定Genie应用已经开发完毕并准备布署, 并且它托管在一个git仓库中.

假如, 由`Genie.Generator.newapp("MyGenieApp")`生成的`MyGenieApp`应用托管在`github.com/user/MyGenieApp`.

下面的脚本在Ubuntu 20.04下运行, 其它Linux版本可能需要稍作修改.

## 在服务器上安装并运行Genie应用

首先安装依赖库

```shell
~> ssh -i "ssh-key-for-instance.pem" user@123.123.123.123
~> git clone github.com/user/MyGenieApp
~> cd MyGenieAp
~> julia 
pkg> activate .
pkg> instantiate
julia> exit()
```

为了让App不会因退出控制台而结束, 通过screen来启动(或者tmux).

```shell
screen -S genie
```

然后设置`GENIE_ENV`环境变量为`prod`:

```shell
export GENIE_ENV=prod
```

启动App:

```shell
./bin/server
```

App跑起来后就可以通过地址`123.123.123.123:8000`访问. 注意要配置App使之不服务静态内容(见`config/env/prod.jl`), 静态内容交给Nginx去处理.

## 安装和配置Nginx服务

Nginx将用于反向代理, 它会侦听请80端口的请求并且把请求重定向到侦听在8000端口的Genie应用.

Nginx还用于服务App的静态文件, 即, 位于`./public`目录下的文件.

最后, 它还可以用于处理HTTPS请求, 还是通过重定向到侦听在8000段偶的Genie应用.

安装Nignx后需要配置, 配置文件为`/etc/nginx/nginx.conf`

```
server {
  listen 80;
  listen [::]:80;

  server_name   test.com;
  root          /home/ubuntu/MyGenieApp/public;
  index         welcome.html;

  location / {
      proxy_pass http://localhost:8000/;
  }

  location /css/genie {
      proxy_pass http://localhost:8000/;
  }
  location /img/genie {
      proxy_pass http://localhost:8000/;
  }
  location /js/genie {
      proxy_pass http://localhost:8000/;
  }
}
```

同时, Genie的`server_handle_static_file`也要设置成`false`.

配置好后启动Nginx服务.

## 启用HTTPS

启用HTTP需要域名证书. 一个切实可行的方法是使用[certbot](https://certbot.eff.org/).

安装好certbot后, 使用cerbot工具, 运行:

```shell
sudo certbot --nginx
```

注意这一步会检查nginx配置文件中的域名.