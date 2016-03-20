---
title: 压力测试脚本
description: >
  利用curl，自己写的服务器压力测试脚本
layout: post
categories: 运维
---

做压力测试一直是用JMeter，最近自己写了一个脚本来进行一些简单的压力测试，特此记录，以备查询。

工具-curl
---

curl是一种命令行工具，作用是发出网络请求，然后得到和提取数据，显示在"标准输出"（stdout）上面。它支持多种协议：FTP, FTPS, HTTP, HTTPS, GOPHER, TELNET, DICT, FILE 以及 LDAP。

需求
---

利用curl模拟用户登录，因此需要记录登录时的cookie，并用此cookie在登录后请求一个只有登录用户才能访问的页面。

命令格式
---

用户登录命令：

```
curl -c ck.txt -X POST -H "Content-type: multipart/form-data;charset=UTF-8;boundary=----------------------------4ebf00fbcf09" --data-binary @formFilePath url

-c 表示将返回的cookie保存于ck.txt中
-X 表示提交方法采用POST
-H 表示头部信息
--data-binary 表示提交的数据存放于文件formFilePath中，注意@不能忘，表示提交的数据存放在文件中
```

下面仔细说一下-H的内容，我在Content-type中说明了表单提交的格式为multipart/form-data，正式这个格式说明，折腾了我将近一天。

命令调整
---

大家都知道当表单需要上传文件时，才需要指定enctype="multipart/form-data"。如果没有指定，浏览器默认采用的是application/x-www-form-urlencoded。事实上，在模拟用户登录才作时，我并没由提交文件，所以不需要用multipart/form-data格式，当我采用application/x-www-form-urlencoded格式时，登录操作轻轻松松就完成了，但是当我采用multipart/form-data格式时，却返回404 Not Found错误。结合实际项目，我定位到出现这个错误的原因是我提交的数据没有通过验证。接着，我就开始调整我提交的数据的格式。

下面讲一下，multipart/form-data提交数据的格式。

```
POST url HTTP/1.1
User-Agent: curl/7.22.0 (i686-pc-linux-gnu) libcurl/7.22.0 OpenSSL/1.0.1 zlib/1.2.3.4 libidn/1.23 librtmp/2.3
Host: ServerIP:port
Accept: */*
Content-type: multipart/form-data;charset=UTF-8;boundary=----------------------------4ebf00fbcf09
Content-Length: 0
------------------------------4ebf00fbcf09
Content-Disposition: form-data; name="role"

￥%&*
------------------------------4ebf00fbcf09
Content-Disposition: form-data; name="account"

account
------------------------------4ebf00fbcf09
Content-Disposition: form-data; name="psw"

1
------------------------------4ebf00fbcf09--
```

上述格式中，boundary=----------------------------4ebf00fbcf09是用来分隔不同的表单数据的。
以上格式是非常严格的，必须一字不差，我就是因为格式有偏差，所以折腾了好久。浏览器会自动帮我们按照标准格式发送请求，可是curl的格式需要自己手动调整。
POST，User-Agent，Host，Accept，Content-Length的内容，curl会帮我们生成，Content-type的内容由-H制定，一开识我没有加boundary，所以一直收到404错误。
Content-Length后面跟的是表单数据，我把表单数据放在文件中。

```
    第一行是boundary，注意行为的换行，因该是\r\n，\r表示光标移到行首，\n表示下一行，有的操作系统会只采用\n来表示换行，这样的话格式就会出错，我采用的是jEdit编辑器，该编辑器可以指定换行采用\r\n还是\n.
    第二行指定表单数据的参数名，接着换行。
    第四行还是换行，这一行不能省。
    第五行是参数值，不需要隐号，直接写上值，这里我写的是￥%&*，表示乱码。
    接下来又是一个boundary，表示另一个表单字段。
    ......
    尾行boundary后面跟了额外的--，这个不能少。
```

下面说一下乱码的问题。
---

我在Content-type中指定charset=UTF-8，一般浏览器会帮我们把数据转成这个指定的编码格式后发送，但是curl没有这个功能。而我测试项目在服务器端，由毫不讲理的进行了request.setCharset("GBK")的操作，所以，事实上，此处Content-type中的charset=UTF-8是不起作用的，所以服务器端接收到的数据一直是乱码。之后我将jdit编辑器的编码格式设为GBK，发送请求，登录成功，返回状态码302，我又在curl命令的最后添加-L，表示跟随重定向，终于返回200的状态码。

碰到的http状态码
---

```
100-199 用于指定客户端应相应的某些动作。 
200-299 用于表示请求成功。 
300-399 用于已经移动的文件并且常被包含在定位头信息中指定新的地址信息。 
400-499 用于指出客户端的错误。 
500-599 用于支持服务器错误。
```

200：(SC_OK)的意思是一切正常。一般用于相应GET和POST请求。这个状态码对servlet是缺省的；如果没有调用setStatus方法的话，就会得到200。

404：表示请求的页面没有找到；

301 (Moved Permanently) 

301 (SC_MOVED_PERMANENTLY)状态是指所请求的文档在别的地方；文档新的URL会在定位响应头信息中给出。浏览器会自动连接到新的URL。 

302 (Found/找到) 

与301有些类似，只是定位头信息中所给的URL应被理解为临时交换地址而不是永久的。注意：在 HTTP 1.0中，消息是临时移动(Moved Temporarily)的而不是被找到，因此HttpServletResponse中的常量是SC_MOVED_TEMPORARILY不是我们以为的SC_FOUND。 

```
注意 :代表状态码302的常量是SC_MOVED_TEMPORARILY而不是SC_FOUND。 
```

状态码302是非常有用的因为浏览器自动连接在定为响应头信息中给出的新URL。这非常有用，而且为此有一个专门的方法——sendRedirect。使用response.sendRedirect(url)比调用response.setStatus(response.SC_MOVED_TEMPORARILY)和response.setHeader("Location", url)多几个好处。首先，response.sendRedirect(url)方法明显要简单和容易。第二，servlet自动建立一页保存这一连接以提供给那些不能自动转向的浏览器显示。最后，在servlet 2.2版本（J2EE中的版本）中，sendRedirect能够处理相对路径，自动转换为绝对路径。但是你只能在2.1版本中使用绝对路径。 
如果你将用户转向到站点的另一页中，你要用 HttpServletResponse 中的 encodeURL 方法传送URL。这么做可预防不断使用基于URL重写的会话跟踪的情况。URL重写是一种在你的网站跟踪不使用 cookies 的用户的方法。这是通过在每一个URL尾部附加路径信息实现的，但是 servlet 会话跟踪API会自动的注意这些细节。会话跟踪在第九章讨论，并且养成使用 encodeURL 的习惯会使以后添加会话跟踪的功能更容易很多。 

```
核心技巧 
如果你将用户转向到你的站点的其他页面，用 response.sendRedirect(response.encodeURL(url)) 的方式事先计划好会话跟踪(session tracking)要比只是调用 response.sendRedirect(url) 好的多。 
这个状态码有时可以与301交换使用。例如，如果你错误的访问了http://host/~user（路径信息不完整），有些服务器就会回复301状态码而有些则回复302。从技术上说，如果最初的请求是GET浏览器只是被假定自动转向。如果想了解更多细节，请看状态码307的讨论。 
```

307 (Temporary Redirect/临时重定向) 

浏览器处理307状态的规则与302相同。307状态被加入到 HTTP 1.1中是由于许多浏览器在收到302响应时即使是原始消息为POST的情况下仍然执行了错误的转向。只有在收到303响应时才假定浏览器会在POST请求时重定向。添加这个新的状态码的目的很明确：在响应为303时按照GET和POST请求转向；而在307响应时则按照GET请求转向而不是POST请求。注意：由于某些原因在HttpServletResponse中还没有与这个状态对应的常量。该状态码是新加入HTTP 1.1中的。 

```
注意: 在 HttpServletResponse 中没有 SC_TEMPORARY_REDIRECT 常量，所以你只能显示的使用307状态码。
```

