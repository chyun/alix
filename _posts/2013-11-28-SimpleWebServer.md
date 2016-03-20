---
title: A Simple Web Server
description: >
  how tomcat works 1 A Simple Web Server
layout: post
categories: tomcat learning
---

how tomcat works: 1 A Simple Web Server!
=====================

HTTP Requests
--------
An HTTP request consists of three components:

> 
> - Method-Uniform Resource Identifier (URI)-Protocol/Version
> - Request headers
> - Entity body.

An example of an HTTP request is the following:

```sh
POST /examples/default.jsp HTTP/1.1                                  //method uri protocol/version
                                                                     //blank line
Accept: text/plain; text/html
Accept-Language: en-gb
Connection: Keep-Alive
Host: localhost
User-Agent: Mozilla/4.0 (compatible; MSIE 4.01; Windows 98)
Content-Length: 33
Content-Type: application/x-www-form-urlencoded
Accept-Encoding: gzip, deflate
                                                                     //blank line
lastName=Franks&firstName=Michael                                    //entity body
```
HTTP Responses
--------
An HTTP request consists of three components:

> 
> - Protocol-Status code-Description
> - Response headers
> - Entity body.

An example of an HTTP response is the following:

```
HTTP/1.1 200 OK 
Server: Microsoft-IIS/4.0 
Date: Mon, 5 Jan 2004 13:13:33 GMT 
Content-Type: text/html 
Last-Modified: Mon, 5 Jan 2004 13:13:12 GMT 
Content-Length: 112 
<html>
<head>
<title>HTTP Response Example</title>
</head>
<body>
Welcome to Brainy Software
</body>
</html>
```

HTTP协议的主要特点可概括如下：
----

1. 支持客户/服务器模式。
2. 简单快速：客户向服务器请求服务时，只需传送请求方法和路径。请求方法常用的有GET、HEAD、POST。每种方法规定了客户与服务器联系的类型不同。由于HTTP协议简单，使得HTTP服务器的程序规模小，因而通信速度很快。
3. 灵活：HTTP允许传输任意类型的数据对象。正在传输的类型由Content-Type加以标记。
4. 无连接：无连接的含义是限制每次连接只处理一个请求。服务器处理完客户的请求，并收到客户的应答后，即断开连接。采用这种方式可以节省传输时间。
5. 无状态：HTTP协议是无状态协议。无状态是指协议对于事务处理没有记忆能力。缺少状态意味着如果后续处理需要前面的信息，则它必须重传，这样可能导致每次连接传送的数据量增大。另一方面，在服务器不需要先前信息时它的应答就较快。


请求方法（所有方法全为大写）有多种，各个方法的解释如下：
---
```
GET     请求获取Request-URI所标识的资源
POST    在Request-URI所标识的资源后附加新的数据
HEAD    请求获取由Request-URI所标识的资源的响应消息报头
PUT     请求服务器存储一个资源，并用Request-URI作为其标识
DELETE  请求服务器删除Request-URI所标识的资源
TRACE   请求服务器回送收到的请求信息，主要用于测试或诊断
CONNECT 保留将来使用
OPTIONS 请求查询服务器的性能，或者查询与资源相关的选项和需求 
```

通讯利用socket
