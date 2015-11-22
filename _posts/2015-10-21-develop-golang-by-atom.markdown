---
layout:     post
title:      "使用Atom开发golang"
subtitle:   "在Windows上使用Atom编辑器开发golang"
date:       2015-10-21 21:00:00
author:     "刘江湖"
header-img: "img/post-bg-01.jpg"
tags:
    - golang
---

之前一直写go是用sublime text， 但sublime text有个不是很舒服的地方是没有办法跳转实例对象的方法上，只能跳转到package的type或function.  LiteIde倒是可以实现这功能，但相对来说，页面比较丑，逼格不够。-_-

偶然发现Atom这个编辑器，发现还是比较漂亮的，配合各种插件，写golang也比较舒服,就特意把配置环境的步骤记录下来。

### Go环境安装
去golang.org上下载安装包（需要翻墙），然后安装，完成后会自动在系统环境变量里面加上go\bin。
然后建立一个文件夹，作为gopath的开发路径。 然后在系统环境变量里面，新建一个GOPATH, 指向此目录。
在此目录下再新建3个目录，  bin, src, pkg。

### 安装Atom
去atom.io上下载windows的安装包，直接运行就可以了。

### 下载package

* 安装mercurial.

* 安装golint

```shell
go get github.com/golang/lint
go install github.com/golang/lint
go install github.com/golang/lint/golint
```

* 安装gooracle

```shell

go get code.google.com/p/go.tools/cmd/oracle
go install code.google.com/p/go.tools/cmd/oracle
```

* 安装goimport

```shell
go get golang.org/x/tools/cmd/goimports
go install golang.org/x/tools/cmd/goimports
```

* 安装gocode

```shell
go get -u github.com/nsf/gocode
go install -u github.com/nsf/gocode
```
* 安装 godef

```shell
go get -v code.google.com/p/rog-go/exp/cmd/godef
go install -v code.google.com/p/rog-go/exp/cmd/godef
```

* go install完后会在$GOPATH/bin, 也就是之前建立的目录出现各种工具：


1. gocode 提供代码补全
2. godef 代码跳转
3. gofmt 自动代码整理
4. golint 代码语法检查
5. goimports 自动整理imports
6. oracle 代码callgraph查询

* 最后的配置结果(安装完之后 Package->Go Plus->Display Go Information), 要是没有出现红色的，就比较OK了。

### 安装Atom的插件

1. 在Settings的Install中，输入go-plus， 选择安装
2. 输入atom-terminal-panel， 安装。
    因为Atom没有编译工具，所以通过这个命令段来执行命令进行编译。 ctrl + ~ 就可以呼出这个终端，然后在里面输入go命令。
3. go-plus的默认跳转快捷键在windows下不起作用，所以需要修改： 在settings种选择open config folder, 在packages下找到go-plus, 修改keymaps的golang:godef 快捷键。

### 总结
至此大功告成，可以享受Atom开发golang的便捷性了。
注： 好像visual studio code也支持go的插件的，但目前还不够完善。
