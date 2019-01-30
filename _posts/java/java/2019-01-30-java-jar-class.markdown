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