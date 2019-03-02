---
layout: post
title:  "HTTP学习"
date:   2019-03-01
categories: http
tag: http
---

* content
{:toc}

# HTTP简介 #

HTTP协议是Hyper Text Transfer Protocol(超文本传输协议)的缩写，是用于从万维网（WWW:World Wide Web）服务器传输超文本到本地浏览器的传送协议。

HTTP 是基于 TCP/IP 通信协议来传递数据（ HTML 文件，图片文件，查询结果等）。

# HTTP工作原理 #

1. HTTP 协议工作于客户端-服务端架构上。浏览器作为 HTTP 客户端通过 URL 向 HTTP 服务端即 WEB 服务器发送所有请求的载体。
2. Web 常见的服务器有： Apache 服务器，IIS 服务器(Internet Information Services) 等。
3. Web 服务器根据接收到的请求，向客户端发送响应消息。
4. HTTP 默认端口号为80，但是我们也可以修改成8080后者其他端口。

**HTTP 三点主要事项：**

> * HTTP是无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户端请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
> 
> * HTTP是媒体独立的：这意味着，只要客户端和服务端知道如何处理的数据内容，任何类型的数据都可以通过HTTP发送。客户端以及服务器指定使用适合的MIME-type内容类型
> 
> * HTTP是无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力，缺少状态意味着如果后续处理需要前面的信息，他必须重传，这样可能导致每次传送的数据量增大。另一方面，在服务器不需要先前信息时，他处理应答就比较快。



# HTTP消息结构 #

1. HTTP是基于客户端/服务端（C/S）的架构模型，通过一个可靠的链接来换换信息，是一个无状态的请求/响应协议。
2. 一个HTTP“客户端”是一个应用程序（ Web 浏览器或者其它任何客户端），通过连接到服务器达到向服务器发送一个或多个HTTP请求的目的。
3. 一个HTTP“服务器”同样也是一个应用程序（通常一个 Web 服务，如 Apache Web 服务器或 IIS 服务器等），通过接收客户端的请求并向客户端发送 HTTP 响应数据。
4. HTTP 使用统一资源标识符（ Uniform Resource Identifiers , URI ）来传输数据和建立连接。
5. 一旦建立连接后，数据消息就通过类似 Internet 邮件所使用的格式[RFC5322]和多用途Internet 邮件扩展 ( MIME )[RFC2045]来传送。

## 客户端请求消息 ##

HTTP 报文由三部分组成（请求行 + 请求头 + 请求体），下面给出了请求报文的一般格式

![/images/2019-02/2019-02-28-http-http-study/TL20190302154153.png](/images/2019-02/2019-02-28-http-http-study/TL20190302154153.png)

以下是一个实际的请求报文：

![/images/2019-02/2019-02-28-http-http-study/TL20190302154220.png](/images/2019-02/2019-02-28-http-http-study/TL20190302154220.png)

## 服务器响应消息 ##

HTTP 响应由四个部分组成（ 状态行 + 消息报头 + 空行 + 响应正文 ），下面是一个响应的图片

![/images/2019-02/2019-02-28-http-http-study/TL20190302155226.png](/images/2019-02/2019-02-28-http-http-study/TL20190302155226.png)

# HTTP请求方法 #

根据HTTP标准，HTTP请求可以使用多种请求方法。

HTTP1.0定义了三种请求方法：GET,POST和HEAD方法。

HTTP1.1新增了五种请求方法：OPTIONS，PUT，DELETE，TEACE和CONNECT方法。

序号 | 方法 | 描述
--|--|--
1 | GET | 请求指定的页面信息，并返回实体主体。
2 | HEAD | 类似于get请求，只不过返回的响应中没有具体的内容，用于获取报头
3 | POST | 向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。
4 | PUT | 从客户端向服务器传送的数据取代指定的文档的内容。
5 | DELETE | 请求服务器删除指定的页面。
6 | CONNECT | HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。
7 | OPTIONS | 允许客户端查看服务器的性能。
8 | TRACE | 回显服务器收到的请求，主要用于测试或诊断。

# HTTP响应头信息 #


应答头 | 说明
-- | --
Allow | 服务器支持哪些请求方法（如GET、POST等）。
Content-Encoding | 文档的编码（Encode）方法。只有在解码之后才可以得到Content-Type头指定的内容类型。利用gzip压缩文档能够显著地减少HTML文档的下载时间。Java的GZIPOutputStream可以很方便地进行gzip压缩，但只有Unix上的Netscape和Windows上的IE 4、IE 5才支持它。因此，Servlet应该通过查看Accept-Encoding头（即request.getHeader("Accept-Encoding")）检查浏览器是否支持gzip，为支持gzip的浏览器返回经gzip压缩的HTML页面，为其他浏览器返回普通页面。
Content-Length | 表示内容长度。只有当浏览器使用持久HTTP连接时才需要这个数据。如果你想要利用持久连接的优势，可以把输出文档写入 ByteArrayOutputStream，完成后查看其大小，然后把该值放入Content-Length头，最后通过byteArrayStream.writeTo(response.getOutputStream()发送内容。
Content-Type | 表示后面的文档属于什么MIME类型。Servlet默认为text/plain，但通常需要显式地指定为text/html。由于经常要设置Content-Type，因此HttpServletResponse提供了一个专用的方法setContentType。
Date | 当前的GMT时间。你可以用setDateHeader来设置这个头以避免转换时间格式的麻烦。
Expires | 应该在什么时候认为文档已经过期，从而不再缓存它？
Last-Modified | 文档的最后改动时间。客户可以通过If-Modified-Since请求头提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304（Not Modified）状态。Last-Modified也可用setDateHeader方法来设置。
Location | 表示客户应当到哪里去提取文档。Location通常不是直接设置的，而是通过HttpServletResponse的sendRedirect方法，该方法同时设置状态代码为302。
Refresh | 表示浏览器应该在多少时间之后刷新文档，以秒计。除了刷新当前文档之外，你还可以通过setHeader("Refresh", "5; URL=http://host/path")让浏览器读取指定的页面。 <br/>注意这种功能通常是通过设置HTML页面HEAD区的＜META HTTP-EQUIV="Refresh" CONTENT="5;URL=http://host/path"＞实现，这是因为，自动刷新或重定向对于那些不能使用CGI或Servlet的HTML编写者十分重要。但是，对于Servlet来说，直接设置Refresh头更加方便。 <br>注意Refresh的意义是"N秒之后刷新本页面或访问指定页面"，而不是"每隔N秒刷新本页面或访问指定页面"。因此，连续刷新要求每次都发送一个Refresh头，而发送204状态代码则可以阻止浏览器继续刷新，不管是使用Refresh头还是＜META HTTP-EQUIV="Refresh" ...＞。 <br/>注意Refresh头不属于HTTP 1.1正式规范的一部分，而是一个扩展，但Netscape和IE都支持它。
Server | 服务器名字。Servlet一般不设置这个值，而是由Web服务器自己设置。
Set-Cookie | 设置和页面关联的Cookie。Servlet不应使用response.setHeader("Set-Cookie", ...)，而是应使用HttpServletResponse提供的专用方法addCookie。参见下文有关Cookie设置的讨论。
WWW-Authenticate | 客户应该在Authorization头中提供什么类型的授权信息？在包含401（Unauthorized）状态行的应答中这个头是必需的。例如，response.setHeader("WWW-Authenticate", "BASIC realm=＼"executives＼"")。 
注意Servlet一般不进行这方面的处理，而是让Web服务器的专门机制来控制受密码保护页面的访问（例如.htaccess）。

# HTTP状态码 #

# HTTP内容类型 #

# 参考资料 #

[http://www.runoob.com/http/http-intro.html](http://www.runoob.com/http/http-intro.html)

