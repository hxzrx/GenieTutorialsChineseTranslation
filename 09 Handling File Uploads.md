# 处理文件上传

https://genieframework.com/docs/genie/v5.11/tutorials/Handling-File-Uploads.html

Genie内置支持文件上传, 上传文件(作为`POST`的变量)集可以通过`Requests.filespayload`访问, 或者使用`Requests.filespayload(key)`检索出指定的文件表单对应的数据, 其中, `key`是表单中输入文件的名称.

下例包含两个路由, 第一个路由处理`GET`请求, 显示上传表单, 第二个路由处理`POST`请求, 处理上传, 从上传数据中生成一个文件, 保存, 并显示文件状态.

**提示**: 可以在同一个URL上定义多个路由, 只要它们拥有不同的HTTP方法, 例如`GET`, `POST`.

## 举例说明

```julia
using Genie, Genie.Router, Genie.Renderer.Html, Genie.Requests

form = """
<form action="/" method="POST" enctype="multipart/form-data">
  <input type="file" name="yourfile" /><br/>
  <input type="submit" value="Submit" />
</form>
"""

route("/") do
  html(form)
end

route("/", method = POST) do
  if infilespayload(:yourfile)  # yourfile是form中input的name
    write(filespayload(:yourfile))

    stat(filename(filespayload(:yourfile)))
  else
    "No file uploaded"
  end
end

up()
```

浏览器访问http://127.0.0.1:8000/, 选择一个文件, 点击提交, 页面会显示如下统计信息:

```
StatStruct("The Little Learner.epub" size: 10677366 bytes device: 2194997704 inode: 816405 mode: 0o100666 (-rw-rw-rw-) nlink: 1 uid: 0 gid: 0 rdev: 0 blksz: 4096 blocks: 20856 mtime:  (just now) ctime:  (just now))
```

帮助文档

```
help?> infilespayload
search: infilespayload

  infilespayload(key::Union{String,Symbol}) :: Bool

  Checks if the collection of uploaded files contains a file stored under the
  key name.
```

```
help?> filespayload
search: filespayload infilespayload

  filespayload() :: Dict{String,HttpFile}

  Collection of form uploaded files.

  ────────────────────────────────────────────────────────────────────────────

  filespayload(filename::Union{String,Symbol}) :: HttpFile

  Returns the HttpFile uploaded through the key input name.
```

```
help?> stat
search: stat STATUS_CODES lstat @static startswith stacktrace StackTraces

  stat(file)

  Return a structure whose fields contain information about the file. The
  fields of the structure are:

  Name    Description
  ––––––– ––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––––
  desc    The path or OS file descriptor
  size    The size (in bytes) of the file
  device  ID of the device that contains the file
  inode   The inode number of the file
  mode    The protection mode of the file
  nlink   The number of hard links to the file
  uid     The user id of the owner of the file
  gid     The group id of the file owner
  rdev    If this file refers to a device, the ID of the device it refers to
  blksize The file-system preferred block size for the file
  blocks  The number of such blocks allocated
  mtime   Unix timestamp of when the file was last modified
  ctime   Unix timestamp of when the file's metadata was changed
```

