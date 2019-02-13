---
layout: post
title:  "替换jar包中的class文件"
date:   2019-01-30 16:40:39 +0800
categories: java
tag: jar
---

* content
{:toc}

# 解压jar包 #

- 直接利用winwar解压jar包，解压包的路径与jar包是同级路路径
![png](/images/2019-01/2019-01-30-java-jar-class/TL20190130165319.png)


# 替换你需要修改的class文件 #

- 利用重新编译后的`Test.class`文件替换`BOOT-INF/classes/com/yangl/test/web/controller/Test.class`中的class文件

# 重新压缩该class文件 #

利用如下命令将class文件进行打包

    jar -uvf test-1.0.0.0-SNAPSHOT.jar BOOT-INF/classes/com/yangl/test/web/controller/Test.class

> Test.class文件路径要与他在jar中存在的路径一致，如果不一致，他将会被压缩到你实际存在文件的路径

# 参数解释 #
下面是从百度搜索的jar命令各个参数的详细介绍

    -c  建立新的归档
    -t  列出归档的目录
    -x  从归档中撷取已命名的 (或所有) 档案
    -u  更新现有归档
    -v  在标准输出中产生详细输出
    -f  指定归档档案名称
    -m  包含指定清单档案中的清单资讯
    -e  为独立应用程式指定应用程式进入点已随附於可执行 jar 档案中
    -0  仅储存；不使用 ZIP 压缩方式
    -M  不为项目建立清单档案
    -i  为指定的 jar 档案产生索引资讯
    -C  变更至指定目录并包含後面所列的档案