[TOC]

# 工具打包


我们可以通过`gf`命令行工具的`pack`命令实现对任意文件/目录的打包，关于`gf`命令行工具的安装和使用具体请查看[CLI工具链](toolchain/cli.md)章节。由于通过命令行工具进行打包比较简便，因此也是推荐的打包方式。

`gf pack`的命令介绍如下：
```
$ gf pack -h
USAGE
    gf pack SRC DST

ARGUMENT
    SRC  source path for packing, which can be multiple source paths.
    DST  destination file path for packed file. if extension of the filename is ".go" and "-n" option is given,
         it enables packing SRC to go file, or else it packs SRC into a binary file.

OPTION
    -n, --name      package name for output go file
    -p, --prefix    prefix for each file packed into the resource file

EXAMPLES
    gf pack public data.bin
    gf pack public,template data.bin
    gf pack public,template boot/data.go -n=boot
    gf pack public,template,config resource/resource.go -n=resource
    gf pack public,template,config resource/resource.go -n=resource -p=/var/www/my-app
    gf pack /var/www/public resource/resource.go -n=resource
```


## 1. `gf pack`生成`Go`文件

比较推荐的方式是将`Go`文件直接生成到`boot`启动目录，并设置生成`Go`文件的包名为`boot`，这样该资源文件将会被自动引入到项目中。我们将项目的`config,public,template`三个目录的文件打包到`Go`文件，打包命令为：
```
gf pack config,public,template boot/data.go -n boot
```


生成的`Go`文件内容类似于：
```go
package boot

import "github.com/gogf/gf/os/gres"

func init() {
	if err := gres.Add("1f8b08000000000002ffb45a095c4cebfb7f51d2284a515"); err != nil {
		panic(err)
	}
}
```

可以看到，生成的`Go`文件中通过`gres.Add`方法将资源文件的二进制内容添加到默认的资源管理器中，该方法的参数是压缩过后的十六进制字符串，将会在程序启动的时候做解压并在内存中生成一个文件树对象，便于在运行时快速操作文件。

## 2. 使用打包的`Go`文件

由于项目的`main`入口程序文件会引入`boot`包，例如像这样（`module`名称为`my-app`）：
```go
import (
	_ "my-app/boot"
)
```
随后可以在项目的任何地方使用`gres`模块来访问打包的资源文件。

> 如果使用`GF`推荐的[项目目录结构](start/index.md)，在目录结构中会存在`boot`目录（对应包名也是`boot`），用于程序启动设置。因此如果将`Go`文件生成到`boot`目录下，那么将会被自动编译进可执行文件中。

## 3. 打印资源管理文件列表

可以通过`gres.Dump()`方法打印出当前资源管理器中所有的文件列表，输出内容类似于：
```html
2019-09-15T13:36:28+00:00   0.00B config
2019-07-27T07:26:12+00:00   1.34K config/config.toml
2019-09-15T13:36:28+00:00   0.00B public
2019-06-25T17:03:56+00:00   0.00B public/resource
2018-12-04T12:50:16+00:00   0.00B public/resource/css
2018-12-17T12:54:26+00:00   0.00B public/resource/css/document
2018-12-17T12:54:26+00:00   4.20K public/resource/css/document/style.css
2018-08-24T01:46:58+00:00  32.00B public/resource/css/index.css
2019-05-23T03:51:24+00:00   0.00B public/resource/image
2018-08-20T05:02:08+00:00  24.01K public/resource/image/cover.png
2019-05-23T03:51:24+00:00   4.19K public/resource/image/favicon.ico
2018-08-23T01:44:50+00:00   4.19K public/resource/image/gf.ico
2018-12-04T13:04:34+00:00   0.00B public/resource/js
2019-06-27T11:06:12+00:00   0.00B public/resource/js/document
2019-06-27T11:06:12+00:00  11.67K public/resource/js/document/index.js
2019-09-15T13:36:28+00:00   0.00B template
2019-02-02T09:08:56+00:00   0.00B template/document
2018-12-04T12:49:08+00:00   0.00B template/document/include
2018-12-04T12:49:08+00:00 329.00B template/document/include/404.html
2019-03-06T01:52:56+00:00   3.42K template/document/index.html
...
```
需要注意的是，在使用资源管理器中的资源文件时，需要严格按照其路径进行检索获取。
