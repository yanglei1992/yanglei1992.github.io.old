---
layout: post
title:  "jvm(HotSpot)的垃圾回收器"
date:   2018-12-27 21:30:39 +0800
categories: java
tag: java
---

* content
{:toc}

# 将您重定向的次数过多 #

如果登录拦截器将你多次请求的结果返回同一界面，会报重定向次数过多的错误。我们需要将重复重定向的请求过滤掉。

    if (requestURINoCtxPath.equals("/changePassword") || requestURINoCtxPath.equals("/web-test/changePassword")) {
    	//如果是修改密码页面与修改密码url，则不做验证   如果不新增这个就会报“将您重定向的次数过多”次数过多的错误，界面请求太多导致拦截器转发到修改密码界面的重定向太多
    	flag = true;
    } else {
    	//首次登陆修改密码不能跳到其他页面
    	response.sendRedirect("/web-test/changePassword");
    	flag = false;
    }