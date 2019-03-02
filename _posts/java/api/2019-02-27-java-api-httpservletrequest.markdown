---
layout: post
title:  "Servlet——HttpServletRequest对象详解"
date:   2019-02-27
categories: java
tag: API
---

* content
{:toc}

# HttpServletRequest 简介 #

对象：javax.servlet.http.HttpServletRequest

父接口：javax.servlet.ServletRequest

简介：httpservletrequest公共接口类，继承自ServletRequest。客户端浏览器发出的请求被封装成为一个HttpServletRequest对象。对象包含了客户端请求信息包括请求地址，请求的参数，提交的数据，上传的文件客户端的IP甚至客户端操作系统都包含在棋类。HttpServletResponse继承了ServlerResponse接口，并提供了与Http协议有关的方法，这些方法的主要功能是设置HTTP状态码和管理Cookie。

# HttpServletRequest 的方法 #

## getAuthType() ##

**public String getAuthType()**

返回用于保护 servlet 的验证方案的名称。所有 servlet 容器都支持 basic、form 和 client certificate 验证，并且可能还支持 digest 验证。如果没有验证 servlet，则返回 null。 

返回的值与 CGI 变量 AUTH_TYPE 的值相同。

**return**

返回静态成员 BASIC_AUTH、FORM_AUTH、CLIENT_CERT_AUTH、DIGEST_AUTH 之一（适用于 == 比较）或返回指示验证方案的特定于容器的字符串，如果没有验证请求，则返回 null。


----------

## getCookies() ##

**public Cookie[] getCookies()**

返回包含客户端随此请求一起发送的所有Cookie 对象的数组。如果没有发送任何cookie，则此方法返回null。 

**return**

此请求中包含的所有 Cookie 的数组，如果该请求没有 cookie，则返回 null

用例：

Cookie[] cookies=request.getCookies();//取得返回的Cookie数组

----------

## getDateHeader(String name) ## 

**public long getDateHeader(String name)**

读取指定的报头，然后转换成一个Date值。

以表示 Date 对象的 long 值的形式返回指定请求头的值。与包含日期的头（比如 If-Modified-Since）一起使用此方法。 

返回的日期是自格林威治标准时间 1970 年 1 月 1 日起经过的毫秒数。头名称是不区分大小写的。

如果该请求没有指定名称的头，则此方法返回 -1。如果无法将头转换为日期，则该方法将抛出 IllegalArgumentException。

**name**

指定头名称的 String

**return**

表示头中指定的日期的 long 值，该日期被表示为自格林威治标准时间 1970 年 1 月 1 日起经过的毫秒数，如果请求中不包含指定的头，则返回 -1

**Throws IllegalArgumentException** 

如果无法将头值转换为日期

----------

## getHeader(String name) ##

**public String getHeader(String name)**
 
以 String 的形式返回指定请求头的值。如果该请求不包含指定名称的头，则此方法返回 null。如果有多个具有相同名称的头，则此方法返回请求中的第一个头。头名称是不区分大小写的。可以将此方法与任何请求头一起使用。 

**name**

指定头名称的 String

**return**

包含请求头的值的 String，如果该请求没有该名称的头，则返回 null

----------

## getHeaderNames() ##

**public java.util.Enumeration getHeaderNames()**

返回此请求包含的所有头名称的枚举。如果该请求没有头，则此方法返回一个空枚举。 

一些 servlet 容器不允许 servlet 使用此方法访问头，在这种情况下，此方法返回 null。

**return**

随此请求一起发送的所有头名称的枚举；如果该请求没有头，则返回一个空枚举；如果 servlet 容器不允许 servlet 使用此方法，则返回 null

----------

## getHeaders(String name) ##

**public java.util.Enumeration getHeaders(String name)**

以 String 对象的 Enumeration 的形式返回指定请求头的所有值。大多的情况下，每个报头名称在请求中只出现一次。但是 一些头（比如 Accept-Language）就出现多次，而每次出现都使用不同的值。这时，我们可以使用该方法获取一个Enumeration枚举报头每次出现所对应的值。

如果该请求不包含任何指定名称的头，则此方法返回一个空的 Enumeration。头名称是不区分大小写的。可以将此方法与任何请求头一起使用。

**name**

指定头名称的 String

**return**

包含请求头的值的 Enumeration。如果该请求不包含任何具有该名称的头，则返回一个空枚举。如果容器不允许访问头信息，则返回 null

----------

## getIntHeader(String name) ##

**public int getIntHeader(String name)**

读取指定的报头，然后将返回值转换为int类型。

以 int 的形式返回指定请求头的值。如果该请求没有指定名称的头，则此方法返回 -1。如果无法将头转换为整数，则此方法抛出 NumberFormatException。 

头名称是不区分大小写的。

**name**

指定请求头名称的 String

**return**

表示请求头的值的整数，如果该请求没有此名称的头，则返回 -1

**Throws NumberFormatException**

如果无法将头值转换为 int

----------

## getMethod() ##

**public String getMethod()**

返回谁发出此请求的 HTTP 方法的名称，例如 GET、POST 或 PUT。

返回的值与 CGI 变量 REQUEST_METHOD 的值相同。 

**return**

指定用于发出此请求的方法名称的 String

----------

## getPathInfo() ##

**public String getPathInfo()**

返回与客户端发出此请求时发送的 URL 相关联的额外路径信息。额外路径信息位于 servlet 路径之后但在查询字符串之前，并且将以 "/" 字符开头。 

如果没有额外路径信息，则此方法返回 null。

返回的值与 CGI 变量 PATH_INFO 的值相同。

**return**

由 Web 容器解码的 String，用于指定额外路径信息，这些信息在请求 URL 中跟随在 servlet 路径之后但在查询字符串之前；如果该 URL 没有任何额外路径信息，则返回 null

----------

## getPathTranslated() ##

**public String getPathTranslated()**

返回在 servlet 名称之后但在查询字符串之前的额外路径信息，并将它转换为实际路径。返回的值与 CGI 变量 PATH_TRANSLATED 的值相同。 

如果 URL 没有任何额外路径信息，则此方法返回 null，或者 servlet 容器因为某种原因（比如当从归档文件执行 Web 应用程序时）无法将虚拟路径转换为实际路径。 Web 容器不会解码此字符串。

**return**

指定实际路径的 String，如果 URL 没有额外路径信息，则返回 null

----------

## getQueryString() ##

**public String getQueryString()**

返回包含在请求 URL 中路径后面的查询字符串。如：http://www.baidu.com/baidu?word=苹果&ie=utf-8&tn=98012088_dg 。返回：word=苹果&ie=utf-8&tn=98012088_dg。（）返回？号后面的查询字符串。返回如果 URL 没有查询字符串，则此方法返回 null。返回的值与 CGI 变量 QUERY_STRING 的值相同。 

**return**

包含查询字符串的 String，如果 URL 不包含任何查询字符串，则返回 null。该值不由容器解码。

----------

## getRemoteUser() ##

**public String getRemoteUser()**

如果用户已经过验证，则返回发出此请求的用户的登录信息，如果用户未经过验证，则返回 null。用户名是否随每个后续请求发送取决于浏览器和验证类型。返回的值与 CGI 变量 REMOTE_USER 的值相同。 

**return**

指定发出此请求的用户的登录信息的 String，如果用户登录信息未知，则返回 null

----------

## getRequestedSessionId() ##

**public String getRequestedSessionId()**

返回客户端指定的会话 ID。此值可能不同于此请求的当前有效会话的 ID。如果客户端没有指定会话 ID，则此方法返回 null。 

**return**

指定会话 ID 的 String，如果该请求没有指定会话 ID，则返回 null

----------

## isRequestedSessionIdFromCookie() ##

**public boolean isRequestedSessionIdFromCookie()**

检查请求的会话 ID 是否是作为 cookie 进入的。 

**return**

如果会话 ID 是作为 cookie 进入的，则返回 true；否则返回 false

----------

## isRequestedSessionIdFromURL() ##

**public boolean isRequestedSessionIdFromURL()**

检查请求的会话 ID 是否是作为请求 URL 的一部分进入的。 

**return**

如果会话 ID 是作为 URL 的一部分进入的，则返回 true；否则返回 false

----------

## public boolean isRequestedSessionIdFromUrl() ##

**public boolean isRequestedSessionIdFromUrl()**

检查请求会话 ID 是否作为请求 URL 的一部分

----------

## isRequestedSessionIdValid() ##

**public boolean isRequestedSessionIdValid()**

检查请求的会话 ID 是否仍然有效。 

如果客户端没有指定任何会话 ID，则此方法返回 false。

**return**

如果此请求在当前会话上下文中有一个有效会话 id，则返回 true；否则返回 false

----------

## getRequestURI() ##

**public String getRequestURI()**

返回此请求的 URL 的一部分，从协议名称一直到 HTTP 请求的第一行中的查询字符串。Web 容器不会解码此 String。例如： 

HTTP 请求的第一行

返回的值

POST /some/path.html HTTP/1.1

/some/path.html

GET http://foo.bar/a.html HTTP/1.0

/a.html

HEAD /xyz?a=b HTTP/1.1

/xyz

要使用某个方案和主机重新构造 URL，可使用 HttpUtils#getRequestURL。

**return**

包含 URL 从协议名称一直到查询字符串的那一部分的 String

----------

## getRequestURL() ##

**public StringBuffer getRequestURL()**

重新构造客户端用于发出请求的 URL。返回的 URL 包含一个协议、服务器名称、端口号、服务器路径，但是不包含查询字符串参数。如：http://www.baidu.com/baidu?word=苹果&ie=utf-8&tn=98012088_dg 。返回： ttp://www.baidu.com/baidu。（？号前面的部分）

如果已经使用 javax.servlet.RequestDispatcher#forward 转发了此请求，则重新构造的 URL 中的服务器路径必须反映用于获取 RequestDispatcher 的路径，而不是客户端指定的服务器路径。

因为此方法返回 StringBuffer 而不是字符串，所以可以轻松地修改 URL，例如，可以追加查询参数。

此方法对于创建重定向消息和报告错误很有用。

**return**

包含重新构造的 URL 的 StringBuffer 对象

----------

## getServletPath() ##

**public String getServletPath()**

返回此请求调用 servlet 的 URL 部分。此路径以 "/" 字符开头，包括 servlet 名称或到 servlet 的路径，但不包括任何额外路径信息或查询字符串。返回的值与 CGI 变量 SCRIPT_NAME 的值相同。 

如果用于处理此请求的 servlet 使用 "/*" 模式实现匹配，则此方法将返回一个空字符串 ("")。

**return**

包含将被调用或解码的 servlet 的名称或路径的 String（如请求 URL 中指定的那样），如果用于处理请求的 servlet 使用 "/*" 模式实现匹配，则返回一个空字符串。

----------

## getSession(boolean create) ##

**public HttpSession getSession(boolean create)**

只能在发送任何文档内容到客户程序之前调用这个方法。这个方法会得到一个简单的散列表，用来存储用户的相关数据。如果不存在会话，那么它会创建一个新会话。 

如果 create 为 false 并且该请求没有有效的 HttpSession，则此方法返回 null。

要确保会话得到适当维护，必须在提交响应之前调用此方法。如果容器正使用 cookie 维护会话完整性，并被要求在提交响应时创建新会话，则抛出 IllegalStateException。

**create**

true表示为此请求创建一个新会话（如有必要）；false 表示返回 null（如果没有当前会话）

**return**

与此请求关联的 HttpSession，如果 create 为 false，并且该请求没有有效会话，则返回 null

----------

## getSession() ##

**public HttpSession getSession()**

返回与此请求关联的当前会话，如果该请求没有会话，则创建一个会话。 

**return**

与此请求关联的 HttpSession

----------

## isUserInRole(String role) ##

**public boolean isUserInRole(String role)**

返回一个 boolean 值，指示指定的逻辑“角色”中是否包含经过验证的用户。角色和角色成员关系可使用部署描述符定义。如果用户没有经过验证，则该方法返回 false。 

**role**

指定角色名称的 String

**return**

一个 boolean 值，指示发出此请求的用户是否属于给定角色；如果用户未经过验证，则返回 false

----------

## getUserPrincipal() ##

**public java.security.Principal getUserPrincipal()**

返回包含当前已经过验证的用户的名称的 java.security.Principal 对象。如果用户没有经过验证，则该方法返回 null。 

**return**

包含发出此请求的用户的名称的 java.security.Principal；如果用户没有经过验证，则返回 null

# 参考资料 #

[http://blog.sina.com.cn/s/blog_a10cae760101akwz.html](http://blog.sina.com.cn/s/blog_a10cae760101akwz.html)

[http://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServletRequest.html](http://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServletRequest.html)

