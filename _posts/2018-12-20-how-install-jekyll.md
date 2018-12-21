---
layout: post
title:  "安装Jekyll环境"
date:   2018-12-20 23:21:39 +0800
categories: jekyll
tag: jekyll
---

* content
{:toc}

window下安装jekyll可以分为三步：

1. 安装Ruby
2. 安装DevKit
3. 安装Jekyll
4. 利用Jekyll创建第一个blog
5. 参考资料

----------


## 1、安装Ruby  ##

  我的安装是下载最新版本的`Ruby+Devkit 2.5.3-1 (x64)`[传送门](https:rubyinstaller.org/downloads/)

![install_step](/images/2018-12/pic1.png)

安装完毕后注意环境变量的配置，安装过程中也可以勾选自动添加环境变量

安装后cmd命令窗口执行 `ruby -v`  查看版本信息

![install_step](/images/2018-12/pic2.png)

>安装注意事项：Ruby默认安装自动选择在C盘下，不要修改其默认安装路径
，修改后会导致jekyll安装不成功等一系列异常。



## 2、安装Devkit ##
我的安装是下载最新版本的`Ruby+Devkit 2.5.3-1 (x64)`[传送门](https:rubyinstaller.org/downloads/)


![install_step](/images/2018-12/pic3.png)


文件下载后直接解压开就行（解压目录随意）

cmd 进入解压后的目录,执行一下命令

* ruby dk.rb init
* ruby dk.rb install

安装过程中没有出现error信息就属于正常，由于我这环境已安装完毕，就不截图了

## 3、安装Jekyll ##

先查看你的gem是否安装完毕：

gem -v

安装jekyll

* gem install jekyll

检测是否安装完毕：

* jekyll -v

## 4、利用Jekyll创建第一个blog ##

进入项目目录

* cd tmp

创建博客myblog

* jekyll new myblog (等待两分钟)

![install_step](/images/2018-12/pic5.png)

进入myblog目录

* cd myblog 

启动服务

* jekyll server

![install_step](/images/2018-12/pic6.png)

浏览器输入127.0.0.1:4000可以查看博客

![install_step](/images/2018-12/pic7.png)

## 5、参考资料 ##

[https://www.jekyll.com.cn/](https://www.jekyll.com.cn/)

[https://www.jianshu.com/p/88e3474cef72](https://www.jianshu.com/p/88e3474cef72)

