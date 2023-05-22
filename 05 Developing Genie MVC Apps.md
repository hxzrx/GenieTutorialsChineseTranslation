# 开发MVC应用

https://genieframework.com/docs/genie/v5.11/tutorials/Developing-MVC-Web-Apps.html

本文档使用Genie的app生成器创建一个MVC应用, 包含web页面, REST API断点, 用户验证.

## 起点-创建App

```julia
pkg> add Genie
julia> using Genie
julia> Genie.Generator.newapp_mvc("Watch Tonight", autostart=false)  # Genie会自动去掉空白符, 因此创建的是WatchTonight工程
```

**踩坑**: 创建MVC工程会问选用哪个数据库, 选项3是PostgreSQL, 在windows下编译LibPQ会遇到LLVM问题"Unable to open libLLVM!", 解决方法是, 安装过GitHub的LibPQ.jl: `pkg> add https://github.com/iamed2/LibPQ.jl.git`. 如果仍然报错, 那是因为SearchLight的原因, 单独安装SearchLight应该可以解决`pkg> add SearchLight`. 如果还有提示, 安装[LLVM](https://github.com/llvm/llvm-project/releases), 并且勾选添加路径. 最后, 添加一个路径, 其中包含libpq.dll, 如果本地没有安装PostgreSQL数据库, 可以安装pgAdmin, 其中包含libpq.dll.

注意不要自动开始, 通过设置参数`autostart=false`, 否则会使用默认参数去连接数据库而导致失败. 

工程创建完后会提示:

```
┌ Info: Your new Genie app is ready!
│         Run
│ julia> Genie.loadapp()
│ to load the app's environment
│         and then
│ julia> up()
└ to start the web server on port 8000.
```

随后, 修改数据库配置`./WatchTonight/db/connection.yml`, 注意这个文件默认是只读的!

涉及的几个库: LibPQ, SearchLightPostgreSQL, SearchLight, Genie在WatchTonight目录下打开控制台, 启动Julia, 进入包管理器, 依次add这几个库, 然后激活本目录`"pkg> activate ."`, 退出pkg, 依次using上述几个库, 如果没有报错表示一切OK, 首先运行`Genie.loadapp()`, 然后运行`up()`, 打开主页http://127.0.0.1:8000/看效果. 一般来说, 如果using这四个库都不报错, 则后面都不会出错. 重启后如果又出现LibPQ的问题, 重新从GitHub安装LibPQ就可以解决.

### 工作原理

Genie使用路由的概念, 将URL映射到此app的请求句柄(即一个Julia函数), 打开`routes.jl`文件, 就会发现其中定义了对`"/"`URL请求的路由, 句柄是服务一个静态文件`welcome.html`, 位于`public/`目录下:

```julia
route("/") do
    serve_static_file("welcome.html")
end
```

##  连接数据库

**首先**配置数据库连接, 位于`./WatchTonight/db/connection.yml`

```yml
dev:
  adapter: PostgreSQL
  database: dev
  host: localhost
  username: genie
  password: 
  port: 5432
```

数据库这些都要先建好, 保证数据库连接正常. 若密码留空, 则在启动时会提示输入密码.

**然后**到REPL加载数据库配置:

```julia> include(joinpath("config", "initializers", "searchlight.jl"))```

正常情况下会看到连接成功的文字提示:`PostgreSQL connection (CONNECTION_OK)......`

## 创建资源

*资源*是由App以URL暴露出的实体. 在Genie的MVC应用中, 资源表示层一捆模型(Model), 视图(view), 控制器(Controller)的文件, 并可能有额外的文件, 例如创建数据库表格, 测试, 模型数据验证器等的迁移文件. 这里以创建一个影视文件为例. 到REPL操作:

```julia
julia> Genie.Generator.newresource("movie")
#...\WatchTonight\app\resources\movies\MoviesController.jl
julia> SearchLight.Generator.newresource("movie")
#...\WatchTonight\app\resources\movies\Movies.jl
#...\WatchTonight\db\migrations\2023052201100800_create_table_movies.jl
#...\WatchTonight\app\resources\movies\MoviesValidator.jl
#...\WatchTonight\test\movies_test.jl
```

这两个命令会在`app/`, `db/`, `test/`下创建多个文件来表示这个影视资源. 注意上面日志中生成的文件名.

### 创建数据库迁移表

迁移文件位于`./WatchTonight/db/migration/`, 编辑`xxxxx_create_table_movies.jl`, 注意其中的模板代码可以辅助编码:

```julia
module CreateTableMovies

import SearchLight.Migrations: create_table, column, primary_key, add_index, drop_table

function up()
  create_table(:movies) do
    [
      primary_key()
      column(:type, :string, limit=10)
      column(:title, :string, limit=100)
      column(:directors, :string, limit=100)
      column(:actors, :string, limit=250)
      column(:country, :string, limit=100)
      # 原文档column(:year, :integer, limit = 4)在执行迁移时会报错, 整数没有limit
      column(:year, :integer)
      column(:rating, :string, limit=10)
      column(:categories, :string, limit=100)
      column(:description, :string, limit=1_000)
    ]
  end

  add_index(:movies, :title)
  add_index(:movies, :actors)
  add_index(:movies, :categories)
  add_index(:movies, :description)
  # add_indices(:movies, :column_name_1, :column_name_2)
end

function down()
  drop_table(:movies)
end

end
```

函数up()实际上是SQL的一种DSL, 在执行迁移时会转换成SQL并在数据库中执行, 信息的传播方式是App --> SearchLight --> LibPQ --> PSQL, 数据库的反馈则相反, 由REPL终端显示.

注意在执行时会自动增加一个`SERIAL`类型的`id`列并作为表格的主键

#### 创建迁移表

为了管理App的潜移, 需要通过SearchLight的迁移系统创建数据库表格, 在REPL上操作:

```julia
julia> SearchLight.Migration.init()
```

这会在数据库的`public`下创建一张名为`schema_migrations`的表格. 注意这个表是迁移表, 其中只有一个version字段, 类型为`varchar(30)`.

### 执行潜移

**迁移状态**检查:

```julia
julia> SearchLight.Migration.status()
[ Info: 2023-05-11 10:12:59 SELECT version FROM schema_migrations ORDER BY version DESC
|   | Module name & status                    |
|   | File name                               |
|---|-----------------------------------------|
|   |                 CreateTableMovies: DOWN |
| 1 | 2023051101365435_create_table_movies.jl |
```

返回内容包含一次`DOWN`的迁移, 表示需要去执行这个迁移, 因为前面只是定义了表格但没有执行.

**执行迁移**在REPL:

```julia
julia> SearchLight.Migration.lastup()
```

其结果是, 在`public`下创建`movies`表格. 此时重新检查迁移状态, 发现已经变成`UP`. 并且在`schema_migrations`表中插入了一条版本号记录.

注意`movies`表格中自动增加了一个`SERIAL`类型的`id`列并作为表格的主键.

## MVC中M的创建

表建好后, 就要创建模型文件来管理数据, 模型文件位于`./WatchTonight/app/resources/movies/Movies.jl`

```julia
module Movies

import SearchLight: AbstractModel, DbId
import Base: @kwdef

export Movie # export Movy, SearchLight.Generator.newresource生成器认为Movies的单数是Movy

@kwdef mutable struct Movie <: AbstractModel
  id::DbId = DbId()
  type::String = "Movie"
  title::String = ""
  directors::String = ""
  actors::String = ""
  country::String = ""
  year::Int = 0
  rating::String = ""
  categories::String = ""
  description::String = ""
end

end
```

这个文件的内容是, 把`movies`数据表的字段定义映射成Julia的一个结构体, 这就是**M**odel.

### 与数据库交互

一旦创建好了模型M, 就可以与数据库进行交互. 在REPL:

```julia
julia> using WatchTonight.Movies
julia> m = Movie(title="Test Movie", actors="John Doe, Jane Doe")
Movie
| KEY                 | VALUE              |
|---------------------|--------------------|
| actors::String      | John Doe, Jane Doe |
| categories::String  |                    |
......
```

结果是创建了一个Movie实例.

执行第一行命令可能会报错``ERROR: UndefVarError: `Movies` not defined``, 这可能是编辑`Movies.jl`没有autoload导致, 通过`julia> include(joinpath("app", "resources", "movies", "Movies.jl"))`可以解决.

新创建的这个Movie实例还没有保存到数据库, 这可在REPL上通过`julia> SearchLight.ispersisted(m)`进行查询. 要**保存**这个记录, 可以通过命令`julia > SearchLight.save(m)`实现:

```julia
julia> SearchLight.save(m)
[ Info: 2023-05-11 13:38:24 INSERT INTO movies ( "type", "title", "directors", "actors", "country", "year", "rating", "categories", "description" ) VALUES ( E'Movie', E'Test Movie', E'', E'John Doe, Jane Doe', E'', 0, E'', E'', E'' ) RETURNING id
true
```

其它的数据查询方法:

```julia
julia> SearchLight.count(Movie)
julia> SearchLight.all(Movie)
```

由此可见, (SearchLight)ORM层的作用是实现Genie数据和DB数据的相互兼容.

### 喂数据(Seeding the data)

目的: 将数据导入数据库.

首先创建导入脚本文件`julia> touch(joinpath("db", "seeds", "seed_movies.jl"))`, 编辑:

```julia
using SearchLight, WatchTonight.Movies
using CSV

Base.convert(::Type{String}, _::Missing) = ""
Base.convert(::Type{Int}, _::Missing) = 0
Base.convert(::Type{Int}, s::String) = parse(Int, s)

function seed()
    for row in CSV.Rows(joinpath(@__DIR__, "netflix_titles.csv"), limit=1000)
        m = Movie()
        m.type = row.type
        m.title = row.title
        m.directors = row.director
        m.actors = row.cast
        m.country = row.country
        m.year = parse(Int, row.release_year)
        m.rating = row.rating
        m.categories = row.listed_in
        m.description = row.description
        save(m)
    end
end
```

添加CSV文件依赖: `pkg> add CSV`

下载数据集: `julia> Base.download("https://raw.githubusercontent.com/essenciary/genie-watch-tonight/main/db/seeds/netflix_titles.csv", joinpath("db", "seeds", "netflix_titles.csv"))`

运行加载脚本:

```julia
julia> include(joinpath("db", "seeds", "seed_movies.jl"))
julia> seed()
```

注意, `seed()`的执行包含很多错误, 因为表格限制了文本长度, 而csv文件中很多记录都超出了限制, for一千次最终只导入了969条记录.

## 设置Web页面

**首先**添加*路由*映射, 编辑`routes.jl`, 添加`/movies`的路由.

```julia
using Genie.Router
using WatchTonight.MoviesController

route("/") do
  serve_static_file("welcome.html")
end

route("/movies", MoviesController.index) #  "/movies"由MoviesController.index函数处理
```

**然后**控制器文件`app/resources/movies/MovieController.jl`

```julia
module MoviesController

function index()
  "Welcome to movies list!"
end

end
```

访问http://127.0.0.1:8000/movies看效果. 这时代码已得到HMR.

**进一步**优化这个函数, 随机显示一条影视记录:

```julia
module MoviesController

using Genie.Renderer.Html, SearchLight, WatchTonight.Movies

function index()
  html(:movies, :index, movies=rand(Movie)) // 传递一个随机的Movie实例
end

end
```

这个新的`index()`函数, 将同级的view目录下的**视图文件**`index.jl.html`作为HTML进行渲染, 创建这个视图文件`julia> touch(joinpath("app", "resources", "movies", "views", "index.jl.html"))`并编辑:

```html
<h1 class="display-1 text-center">Watch tonight</h1>
<%
if ! isempty(movies)
  for_each(movies) do movie
    partial(joinpath(Genie.config.path_resources, "movies", "views", "_movie.jl.html"), movie = movie)
  end
else
  partial(joinpath(Genie.config.path_resources, "movies", "views", "_no_results.jl.html"))
end
%>
```

这时可以查看相关的两个帮助

```julia
help?> Genie.Renderer.Html.partial
  partial(path::String; context::Module = @__MODULE__, vars...) :: String

  Renders (includes) a view partial within a larger view or layout file.
```

```julia
help?> Genie.Renderer.Html.html
  Parses the data input as HTML, returning a HTML HTTP Response.
  Arguments
  ≡≡≡≡≡≡≡≡≡≡≡
    •  data::String: the HTML string to be rendered
    •  context::Module: the module in which the variables are evaluated
       (in order to provide the scope for vars). Usually the controller.
    •  status::Int: status code of the response
    •  headers::HTTPHeaders: HTTP response headers
    •  layout::Union{String,Nothing}: layout file for rendering data
```

可见, `partial`函数是将把所引内容渲染到一个更大的视图或布局文件中, `html`则是将输入数据解析成HTML作为HTTP的响应内容, html的第一个参数实际上是一个HTML模板.

**进一步**创建视图片段文件`_movie.jl.html`:

```html
<div class="container" style="margin-top: 40px;">
  <h3><% movie.title %></h3>

  <div>
    <small class="badge bg-primary"><% movie.year %></small> |
    <small class="badge bg-light text-dark"><% movie.type %></small> |
    <small class="badge bg-dark"><% movie.rating %></small>
  </div>

  <h4><% movie.description %></h4>

  <div><strong>Directed by: </strong><% movie.directors %></div>
  <div><strong>Cast: </strong><% movie.actors %></div>
  <div><strong>Country: </strong><% movie.country %></div>
  <div><strong>Categories: </strong><% movie.categories %></div>
</div>
```

**以及**视图片段文件`_no_results.jl.html`:

```html
<h4 class="container">
  Sorry, no results were found for "$(params(:search_movies))"
</h4>
```

### 使用布局文件

加载Twitter Bootstrap CSS库. 由于这会在多个页面中使用, 因此要将它在主布局文件中加载. 这个布局文件就是`./WatchTonight/app/layouts/app.jl.html`,编辑为:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>Genie :: The Highly Productive Julia Web Framework</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5/dist/css/bootstrap.min.css" rel="stylesheet">
  </head>
  <body>
    <div class="container">
    <%
      @yield
    %>
    </div>
  </body>
</html>
```

要注意这个文件默认是只读的!

刷新页面http://127.0.0.1:8000/movies会看到随机的影视信息.

## 添加搜索特性

目标: 向页面添加搜索表单. 

**首先**编辑`./WatchTonight/app/resources/movies/views/index.jl.html`, 添加一个`div`:

```html
<h1 class="display-1 text-center">Watch tonight</h1>

<div class="container" style="margin-top: 40px;">
  <form action="$( Genie.Router.linkto(:search_movies) )">
    <input class="form-control form-control-lg" type="search" name="search_movies" placeholder="Search for movies and TV shows" />
  </form>
</div>

<%
if ! isempty(movies)
  for_each(movies) do movie
    partial(joinpath(Genie.config.path_resources, "movies", "views", "_movie.jl.html"), movie = movie)
  end
else
  partial(joinpath(Genie.config.path_resources, "movies", "views", "_no_results.jl.html"))
end
%>
```

上述代码向HTML中添加一个`<form>`, 则会通过GET请求提交一个查询.

**接下来**, 添加路由, `routes.jl`:

```julia
route("/movies/search", MoviesController.search, named = :search_movies)
```

最后, 到`./WatchTonight/app/resources/movies/MoviesControllers.jl`中添加句柄函数`search()`

```julia
module MoviesController

using Genie, Genie.Renderer, Genie.Renderer.Html, SearchLight, WatchTonight.Movies

function index()
  html(:movies, :index, movies=rand(Movie))
end

function search()
  isempty(strip(Genie.params(:search_movies))) && Genie.Renderer.redirect(:get_movies)

  movies = SearchLight.find(Movie, SQLWhereExpression("title LIKE ? OR description LIKE ? OR actors LIKE ? OR categories LIKE ?",
    repeat(['%' * Genie.params(:search_movies) * '%'], 4)))

  html(:movies, :index, movies=movies)
end

end
```

刷新http://127.0.0.1:8000/movies看效果, 此时可以搜索!

`search()`函数涉及几个库函数, 有必要查看:

```julia
help?> Genie.params
  function params()
  The collection containing the request variables collection.
  
help?> Genie.Renderer.redirect
  Sets redirect headers and prepares the Response. It accepts 3 parameters: 1
  - Label of a Route (to learn more, see the advanced routes section) 2 -
  Default HTTP 302 Found Status: indicates that the provided resource will be
  changed to a URL provided 3 - Tuples (key, value) to define the HTTP request
  header
```

## 构建REST API

**首先**, 向`routes.jl`添加一个路由:

```
route("/movies/search_api", MoviesController.search_api)
```

然后向控制器文件`MoviesController.jl`添加句柄函数`search_api`.

```julia
function search_api()
  movies = SearchLight.find(Movie, SQLWhereExpression("title LIKE ? OR description LIKE ? OR actors LIKE ? OR categories LIKE ?",
    repeat(['%' * Genie.params(:search_movies) * '%'], 4)))

  Genie.Renderer.Json.json(Dict("movies" => movies))
end
```

访问http://127.0.0.1:8000/movies/search_api?search_movies=xxxx看效果, xxxx是模糊匹配串, 会返回一个json对象.

## Genie插件

对于站点的限制区域, 添加一个数据库后端的授权验证, 使用`GenieAuthentication`插件.

```
pkg> add GenieAuthentication  # 安装插件库
julia> using GenieAuthentication
julia> GenieAuthentication.install(@__DIR__)  # 安装插件文件, 会创建多个文件或目录
julia> SearchLight.Migration.up("CreateTableUsers")  # 执行数据迁移, 创建users表格

julia> include(joinpath("app", "helpers", "GenieAuthenticationViewHelper.jl"))  # 源文档缺少这行, 导致下一步报, 或者重启app解决
julia> Genie.Generator.newcontroller("Admin", pluralize=false)  # 在app/resources/admin下生成新文件
julia> include(joinpath("plugins", "genie_authentication.jl"))  # 报错, 没有定义句柄函数

julia> using WatchTonight.Users
julia> u = User(email = "admin@admin", name = "Admin", password = Users.hash_password("admin"), username = "admin")  // 创建管理员用户
julia> save!(u)  // 保存到数据库

```

注意GenieAuthentication.install()会在`./WatchTonight/app/resources/`下创建`authentication`和`users`两个目录, 它们与`movies`是平级的, 都是对数据表格的定义和映射. 并且, 在`./WatchTonight/db/migrations`下生成了一张`xxxx_create_table_users.jl`的迁移表.

**接着**上面一系列操作, 之后向`routes.jl`添加路由

```julia
using WatchTonight.AdminController
route("/admin/movies", AdminController.index, named = :get_home)
```

**最后**, 向`./WatchTonight/app/resources/admin/AdminController.jl`添加控制器代码:

```julia
module AdminController

using GenieAuthentication, Genie.Renderer, Genie.Exceptions, Genie.Renderer.Html

function index()
  @authenticated!()     # 这行是关键
  h1("Welcome Admin") |> html
end

end
```

 访问http://127.0.0.1:8000/admin/movies, 会重定向到http://127.0.0.1:8000/login

开放用户注册, 编辑`plugins/genie_authentication.jl`文件, 将后面两行取消注释:

```julia
# UNCOMMENT TO ENABLE REGISTRATION ROUTES

route("/register", AuthenticationController.show_register, named = :show_register)
route("/register", AuthenticationController.register, method = POST, named = :register)
```



