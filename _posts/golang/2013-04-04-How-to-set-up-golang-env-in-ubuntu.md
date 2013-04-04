---
layout: post
category : golang
summary: 新手在ubuntu下如何建立golang开发环境。
tags : [golang, ubuntu, Sublime text2]
---
{% include JB/setup %}

  
最近一直在学习golang, 之前一直是在windows平台上，所以想熟悉下linux环境的开发。  
在ubuntu下，和windows具体都差不多。  

***
*下载[golang](https://code.google.com/p/go/downloads/list).
*启动terminal: 切换到下载包所在的目录，输入`sudo tar -C /usr/local -xzf go1.0.2.linux-amd64.tar.gz` 解压文件到`/usr/local`目下。  
如果提示没有权限，可能需要使用命令：`sudo chmod 777 /usr/local` 赋予权限。
*增加golang的环境变量: `sudo gedit ~/.bashrc` 打开profile文件，添加`export PATH=$PATH:/usr/local/go/bin`后保存退出。
*输入go env，应该可以看到golang的一些具体信息了。  

***
golang的环境已经装好了，下面配置开发环境。  
我使用的是`sublime text 2 + gosublime + margo`。 前者是一个免费的，而很漂亮的文本编辑器。
*下载[sulblime text](http://www.sublimetext.com/2), 解压.
*启动sublime text 2，安装package control: 按下control + ~ 调出Console,输入以下代码并回车  
``import urllib2,os;pf='Package Control.sublime-package';ipp=sublime.installed_packages_path();os.makedirs(ipp) if not os.path.exists(ipp) else None;open(os.path.join(ipp,pf),'wb').write(urllib2.urlopen('http://sublime.wbond.net/'+pf.replace(' ','%20')).read())``
*重启 Sublime Text 2，如果在 Preferences -> Package Settings中见到Package Control这一项，就说明安装成功了。
*安装gosublime包：按下control + shift + P键调出命令面板，输入`install`调出`install package`选择并回车, 在之后的弹出框里输入gosublime， 并回车。安装完成后重启. 如果不想手动建立src、pkg、bin等目录，可以再安装gobuild包，之后在project下会多出个create go project， 这样就可以自动帮你创建上述几个目录。
*下载[gocode](https://github.com/DisposaBoy/GoSublime)包实现智能提示功能, 下载[Margo](https://github.com/DisposaBoy/MarGo)实现自动整理代码功能。使用方法:<br/>
如果下载2个文件均是压缩包的话，解压，打开terminal， 并切换至相应目录，输入`go build`， 编译完成后会多个可执行文件，把执行文件拷至/usr/local/go/bin目录下就可以使用了。
*使用：在sublime text新建立一个.go文件，输入一些命令，按下control + B键，输入go build，则进行编译。

***
更多的安装和使用说明，可参考[官方文档](http://golang.org/doc/install)