---
layout: post
title:  "Servlet——ServletRequest对象详解"
date:   2019-02-27
categories: java
tag: API
---

* content
{:toc}

# ServletRequest 简介 #

对象：javax.servlet.ServletRequest

简介：ServletRequest由Servlet容器来管理，当客户请求到来时，容器创建一个ServletRequest对象，封装请求数据，同时创建一个ServletRespoonse对象，封装响应数据。这两个对象将被容器作为service（）方法的参数传递给Servlet，Servlet利用ServletRequest对象传递给客户端发来的请求数据，利用ServletResponse对象发送相应数据。

# ServletRequest 方法 #

## public Object getAttribute(String name) ##

**getAttribute(String name)**

以 Object 形式返回指定会话的值，如果不存在给定名称的会话，则返回 null。 HttpSession对象存储于服务器，并不在网络上来回传送。只有用户请求时才与之关联在一起。些HttpSession以内建的数据结构（散列表）存在，它们都以键和值一一对应存在。

我们可以通过getAttribute（）取得前些时候存储的值。因为，返回的是Object类型，所以，我们要转换成需要的类型。

属性名称应遵守与包名称相同的命名约定。此规范保留匹配 java.*、javax.* 和 sun.* 的名称。

**name**

指定属性名称的 String

**return**

包含属性值的 Object，如果属性不存在，则返回 null

----------

## public java.util.Enumeration getAttributeNames() ##

大多情况下我们都预先知道具体的属性名，从而使用已知的属性名找出它相关联的值。但是，有时我们可以调用这个方法得出给定会话中的所有属性名。如果该请求没有可用的属性，则此方法返回一个空的 Enumeration。 

return

包含请求的属性名称的字符串的 Enumeration

----------

## setCharacterEncoding(String env) ##

**public void setCharacterEncoding(String env) throws java.io.UnsupportedEncodingException**

重写此请求正文中使用的字符编码的名称。必须在使用 getReader() 读取请求参数或读取输入之前调用此方法。否则，此方法没有任何效果。 

**env**

包含字符编码名称的 String。

**Throws java.io.UnsupportedEncodingException**

如果此 ServletRequest 仍然处于可以设置字符编码的状态，但指定编码无效

----------

## getCharacterEncoding() ##

**public String getCharacterEncoding()**

返回此请求正文中使用的字符编码的名称。如果该请求未指定字符编码，则此方法返回 null 

**return**

包含字符编码的名称的 String，如果该请求未指定字符编码，则返回 null

----------

## getContentLength() ##

**public int getContentLength()**

返回Content-Lingth报头的值（作为一个int值返回）。

返回请求正文的长度（以字节为单位），并使输入流可以使用它，如果长度未知，则返回 -1。对于 HTTP servlet，返回的值与 CGI 变量 CONTENT_LENGTH 的值相同。 

**return**

包含请求正文长度的整数，如果长度未知，则返回 -1

----------

## getContentType() ##

**public String getContentType()**

返回Content-Type报头的值（作为一个String值返回）。

返回请求正文的 MIME 类型，如果该类型未知，则返回 null。对于 HTTP servlet，返回的值与 CGI 变量 CONTENT_TYPE 的值相同。 

**return**

包含请求的 MIME 类型名称的 String，如果该类型未知，则返回 null

----------

## getInputStream() ##

**public ServletInputStream getInputStream() throws java.io.IOException**

使用 ServletInputStream 以二进制数据形式获取请求正文。可调用此方法或 #getReader 读取正文，而不是两种方法都调用。 

**return**

包含请求正文的 ServletInputStream 对象

**Throws IllegalStateException** 

如果已经为此请求调用 #getReader 方法

**Throws java.io.IOException**

如果发生输入或输出异常

----------

## getLocalAddr() ##

**public String getLocalAddr()**

返回接收请求的接口的 Internet Protocol (IP) 地址。 

**return**

包含接收请求的 IP 地址的 String。

----------

## getLocale() ##

**public java.util.Locale getLocale()**

基于 Accept-Language 头，返回客户端将用来接受内容的首选 Locale。如果客户端请求没有提供 Accept-Language 头，则此方法返回服务器的默认语言环境。 

**return**

客户端的首选 Locale

----------

## getLocales() ##

**public java.util.Enumeration getLocales()**

返回 Locale 对象的 Enumeration，这些对象以首选语言环境开头，按递减顺序排列，指示基于 Accept-Language 头客户端可接受的语言环境。如果客户端请求没有提供 Accept-Language 头，则此方法返回包含一个 Locale 的 Enumeration，即服务器的默认语言环境。 

**return**

客户端首选 Locale 对象的 Enumeration

----------

## getLocalName() ##

**public String getLocalName()**

返回接收请求的 Internet Protocol (IP) 接口的主机名。 

**return**

包含接收请求的 IP 的主机名的 String。

----------

## getLocalPort() ##

**public int getLocalPort()**

返回接收请求的接口的 Internet Protocol (IP) 端口号。 

**return**

指定端口号的整数

----------

## getParameter(String name) ##

**public String getParameter(String name)**

以 String 形式返回请求参数的值，如果该参数不存在，则返回 null。请求参数是与请求一起发送的额外信息。对于 HTTP servlet，参数包含在查询字符串或发送的表单数据中。 

只有在确定参数只有一个值时，才应该使用此方法。如果参数可能拥有一个以上的值，则使用 #getParameterValues。

如果将此方法用于一个有多个值的参数，则返回的值等于 getParameterValues 返回的数组中的第一个值。

如果参数数据是在请求正文中发送的，比如发生在 HTTP POST 请求中，则通过 #getInputStream 或 #getReader 直接读取正文可能妨碍此方法的执行。

**name**

指定参数名称的 String

**return**

表示单个参数值的 String

用例：

1、在form 表单中有个组件，有个属性name=”Param1”

<</span>input type="text" name="Param1">

2、在Servlet 中使用getParameter(“属性名”)来取得1、中指定属性名组件的值

request.getParameter("Param1")

----------

## getParameterMap() ##

**public java.util.Map getParameterMap()**

request.getParameterMap()的返回类型是Map 类型的对象，也就是符合key-value 的对应关系，

但这里要注意的是，value 的类型是String[]数组,而不是String.。通过它可以将request 请求中的参数

和值变成一个map，如的参数和值：

打印出来，形成的map 结构：map(key,value[])，即：key 是String 型，value 是String[ ]型数组。

例如：request 中的参数t1=1&t1=2&t2=3

形成的map 结构：

key=t1;value[0]=1,value[1]=2//注意t1有两个。

key=t2;value[0]=3

如果直接用map.get("t1"),得到的将是:Ljava.lang.String; value 只所以是数组形式，就是防止参数名有

相同的情况。 

**return**

将参数名称作为键、参数值作为映射值的不可变 java.util.Map。参数映射中的键的类型为 String。参数映射中的值的类型为 String 数组。

用例：

1、在from 表单(注意表单中有同名的name 属性)

用户名：<</span>input type="text" name="name">

密码：<</span>input type="text" name="pwd">

重复密码：<</span>input type="text" name="pwd">

2、getParameterMap 的简单用法

Map map=request.getParameterMap();

out.println(

((String[])map.get("name"))[0]

);

out.println(

((String[])map.get("pwd"))[0]

);

注：可以看到在使得pwd 属性值时出现问题(只能使得第一个pwd 的值。即第一次密码)。

3、getParameterMap 的防止属性名相同的用法

----------

## getParameterNames() ##

**public java.util.Enumeration getParameterNames()**

返回包含此请求中所包含参数的名称的String 对象的Enumeration。如果该请求没有参数，则此方法返回一个空的Enumeration。getParameterNames 返回的每一项都可以转换成String，并可以用在getParameter 或getParameterValues 的调用中。 

return

String对象的 Enumeration，每个 String 都包含一个请求参数的名称；如果该请求没有参数，则返回一个空的 Enumeration

用例：

Enumeration enu=request.getParameterNames();//取得所有的参

数名

while(enu.hasMoreElements())//测试此枚举是否包含更多元素。

{

String Name=(String)enu.nextElement();//接收过来的是个

对象，需要把它转化为字符串。nextElement()取得每个参数名

out.println(Name);

String[]

paraValues=request.getParameterValues(Name);//nextElement 取得的

参数名作为getParameterValues参数，得到的是一个数组

if(paraValues.length==1)//如果数组中只有一项，并且是空字

符串。表示参数没有值

{

String ParamValue=paraValues[0];

if(ParamValue.length()==0)

out.println("没有值");

else

out.println(ParamValue);

}

else

{

//如果数组中有多个值，那么输出每一个值

for(int i=0;ilength;i++)

out.println(paraValues[i]);

}

}

----------

## getParameterValues(String name) ##

**public String[] getParameterValues(String name)**

返回包含给定请求参数拥有的所有值的String 对象数组，如果该参数不存在，则返回null。

它一般用于单选框或复选框组件。

如果该参数只有一个值，则数组的长度为1。

如果同一参数名有可能在表单数据中出现多次，则应该调用getParameterValues(返回客串的数

组)。

**name**

包含请求其值的参数的名称的 String

**return**

包含参数值的 String 对象数组

用例：

1、在from 表单中有一复选框

JSP：<</span>input type="checkbox" name="mynames" value="JSP"><</span>br>

NET：<</span>input type="checkbox" name="mynames" value="NET"><</span>br>

HP：<</span>input type="checkbox" name="mynames" value="HP"><</span>br>

2、使用getParameterValues(“属性名”)返回数据，然再遍历数组

String names[]=request.getParameterValues("mynames");//使用

getParameterValues取得多个相同属性名的值，这些值组成了数组

if(names!=null)

{

int size=java.lang.reflect.Array.getLength(names);//取得数组

的长度

for(int i=0;i

{

out.println(names[i]+"\n");

}

}

----------

## getProtocol() ##

**public String getProtocol()**

返回请求行的第三部分，一般为HTTP/1.0或HTTP/1.1。Servlet一般应该在指定专用于HTTP1.1的响应报头。对于 HTTP servlet，返回的值与 CGI 变量 SERVER_PROTOCOL 的值相同。 

**return**

包含协议名称和版本号的 String

----------

## getReader() ##

**public java.io.BufferedReader getReader() throws java.io.IOException**

使用 BufferedReader 以字符数据形式获取请求正文。读取器根据正文上使用的字符编码转换字符数据。可调用此方法或 #getInputStream 读取正文，而不是两种方法都调用。 

**return**

包含请求正文的 BufferedReader

**Throws UnsupportedEncodingException** 

如果使用的字符集编码不受支持，并且无法对文本进行解码

**Throws IllegalStateException** 

如果已对此请求调用 #getInputStream 方法

**Throws java.io.IOException**: 

如果发生输入或输出异常

----------

## getRealPath(String path) ##

**public String getRealPath(String path)**

deprecated

从 Java Servlet API 的版本 2.1 起，请改用 ServletContext#getRealPath。

----------

## public String getRemoteAddr()

返回发送请求的客户端或最后一个代理的 Internet Protocol (IP) 地址。对于 HTTP servlet，返回的值与 CGI 变量 REMOTE_ADDR 的值相同。 

return

包含发送请求的客户端的 IP 地址的 String

 

----------
## getRemoteHost() ##

**public String getRemoteHost()**

返回发送请求的客户端或最后一个代理的完全限定名称。如果引擎无法或没有选择解析主机名（为了提高性能），则此方法返回以点分隔的字符串形式的 IP 地址。对于 HTTP servlet，返回的值与 CGI 变量 REMOTE_HOST 的值相同。 

**return**

包含客户端的完全限定名称的 String

----------

## public void setAttribute(String name, Object o) ##

**public void setAttribute(String name, Object o)**

对某个会话设置值可以使用本方法，它会将值和名称关联起来。但是要注意，这个方法会替换掉之前设置的值。此方法常常与 RequestDispatcher 一起使用。 

**name**

指定属性名称的 String

**o**

要存储的 Object

----------

## removeAttribute(String name) ##

**public void removeAttribute(String name)**

从此请求中移除指定的值。此方法不是普遍需要的，因为属性只在处理请求期间保留。 

属性名称应遵守与包名称相同的命名约定。以 java.*、javax.* 和 com.sun.* 开头的名称保留给 Sun Microsystems 使用。

**name**

指定要移除属性的名称的 String

----------

## public int getRemotePort() ##

返回发送请求的客户端或最后一个代理的 Internet Protocol (IP) 源端口。 

**return**

指定端口号的整数

----------

## getRequestDispatcher(String path) ##

**public RequestDispatcher getRequestDispatcher(String path)**

返回一个 RequestDispatcher 对象，它充当位于给定路径上的资源的包装器。可以使用 RequestDispatcher 对象将请求转发给资源，或者在响应中包含资源。资源可以是动态的，也可以是静态的。 

指定的路径名可以是相对的，尽管它无法扩展到当前 servlet 上下文之外。如果该路径以 "/" 开头，那么可以相对于当前上下文根解释它。如果 servlet 容器无法返回 RequestDispatcher，则此方法将返回 null。

此方法与 ServletContext#getRequestDispatcher 的不同在于此方法可以采用相对路径。

**path**

指定指向资源的路径名的 String。如果该路径名是相对的，那么它必须是相对于当前 servlet 的。

**return**

充当位于给定路径上的资源的包装器的 RequestDispatcher 对象，如果 servlet 容器无法返回 RequestDispatcher，则返回 null

----------

## getScheme() ##

**public String getScheme()**

返回用于发出此请求的方案的名称，例如 http、https 或 ftp。不同方案具有不同的构造 URL 的规则，这一点已在 RFC 1738 中注明。 

**return**

包含用于发出此请求的方案的名称的 String

----------

## isSecure() ##

**public boolean isSecure()**

返回一个 boolean 值，指示此请求是否是使用安全通道（比如 HTTPS）发出的。 

**return**

指示请求是否是使用安全通道发出的 boolean 值

----------

## getServerName() ##

**public String getServerName()**

返回请求被发送到的服务器的主机名。它是 Host 头值 ":"（如果有）之前的那部分的值，或者解析的服务器名称或服务器 IP 地址。 

**return**

包含服务器名称的 String

----------

## getServerPort() ##

**public int getServerPort()**

返回请求被发送到的端口号。它是 Host 头值 ":"（如果有）之后的那部分的值，或者接受客户端连接的服务器端口。 

**return**

指定端口号的整数


# 参考资料 #

[http://blog.sina.com.cn/s/blog_a10cae760101asmk.html](http://blog.sina.com.cn/s/blog_a10cae760101asmk.html)

[http://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/ServletRequest.html](http://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/ServletRequest.html)

