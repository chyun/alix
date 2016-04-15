---
title: A Simple Servlet Container
description: >
  how tomcat works 2 A Simple Servlet Container
layout: post
category: develop
---

how tomcat works: 2 A Simple Servlet Container!
=====================

The javax.servlet.Servlet Interface
--------
Have 5 methods

> 
> - public void init(ServletConfig config) throws ServletException
> - public void service(ServletRequest request, ServletResponse response) 
    throws ServletException, java.io.IOException
> - public void destroy()
> - public ServletConfig getServletConfig()
> - public java.lang.String getServletInfo()

HttpServlet version1:

```
Listing 2.2: The HttpServer1 Class's await method  
 
package ex02.pyrmont; 
 
import java.net.Socket; 
import java.net.ServerSocket; 
import java.net.InetAddress; 
import java.io.InputStream; 
import java.io.OutputStream; 
import java.io.IOException; 
 
public class HttpServer1 { 
 
  /** WEB_ROOT is the directory where our HTML and other files reside. 
   *    For this package, WEB_ROOT is the "webroot" directory under the 
   *    working directory. 
   *    The working directory is the location in the file system 
   *    from where the java command was invoked. 
   */ 
  // shutdown command 
  private static final String SHUTDOWN_COMMAND = "/SHUTDOWN"; 
 
  // the shutdown command received 
  private boolean shutdown = false; 
 
  public static void main(String[] args) { 
    HttpServer1 server = new HttpServer1(); 
    server.await(); 
  } 
  public void await() { 
    ServerSocket serverSocket = null; 
    int port = 8080; 
    try { 
     serverSocket =    new ServerSocket(port, 1, 
       InetAddress.getByName("127.0.0.1")); 
    } 
 
    catch (IOException e) { 
      e.printStackTrace(); 
      System.exit(1); 
    } 
 
    // Loop waiting for a request 
    while (!shutdown) { 
      Socket socket = null; 
      InputStream input = null; 
      OutputStream output = null; 
      try { 
        socket = serverSocket.accept(); 
        input = socket.getInputstream(); 
        output = socket.getOutputStream(); 
 
        // create Request object and parse 
        Request request = new Request(input); 
        request.parse(); 
 
        // create Response object 
        Response response = new Response(output); 
        response.setRequest(request); 
 
        // check if this is a request for a servlet or 
        // a static resource 
        // a request for a servlet begins with "/servlet/" 
        if (request.getUri().startsWith("/servlet/")) { 
          ServletProcessor1 processor = new ServletProcessor1(); 
          processor.process(request, response); 
        } 
        else { 
          StaticResoureProcessor processor = 
            new StaticResourceProcessor(); 
          processor.process(request, response); 
        } 
 
        // Close the socket 
        socket.close(); 
        //check if the previous URI is a shutdown command 
        shutdown = request.getUri().equals(SHUTDOWN_COMMAND); 
      } 
      catch (Exception e) { 
        e.printStackTrace(); 
        System.exit(1); 
      } 
    } 
  } 
}
```

ServletProcessor1：

```
Listing 2.5: The StaticResourceProcessor class  
 
package ex02.pyrmont; 
 
import java.io.IOException; 
 
public class StaticResourceProcessor { 
 
  public void process(Request request, Response response) { 
    try { 
      response.sendStaticResource(); 
    } 
    catch (IOException e) { 
      e.printStackTrace(); 
    } 
  } 
} 
 

Listing 2.6: The ServletProcessor1 class  
 
package ex02.pyrmont; 
 
import java.net.URL; 
import java.net.URLClassLoader; 
import java.net.URLStreamHandler; 
import java.io.File; 
 
import java.io.IOException; 
import javax.servlet.Servlet; 
import javax.servlet.ServletRequest; 
import javax.servlet.ServletResponse; 
 
public class ServletProcessor1 { 
 
  public void process(Request request, Response response) { 
    String uri = request.getUri(); 
    String servletName = uri.substring(uri.lastIndexOf("/") + 1); 
    URLClassLoader loader = null; 
    try { 
      // create a URLClassLoader 
      URL[] urls = new URL[1]; 
    URLStreamHandler streamHandler = null; 
    File classPath = new File(Constants.WEB_ROOT); 
    // the forming of repository is taken from the 
    // createClassLoader method in 
    // org.apache.catalina.startup.ClassLoaderFactory 
    String repository = 
    (new URL("file", null, classPath.getCanonicalPath() + 
    File.separator)).toString() ; 
    // the code for forming the URL is taken from 
    // the addRepository method in 
    // org.apache.catalina.loader.StandardClassLoader. 
    urls[0] = new URL(null, repository, streamHandler); 
    loader = new URLClassLoader(urls); 
    }
    catch (IOException e) { 
      System.out.println(e.toString() ); 
    } 
    Class myClass = null; 
    try { 
      myClass = loader.loadClass(servletNam
    } 
    catch (ClassNotFoundException e) { 
      System.out.println(e.toString()); 
    } 
 
    Servlet servlet = null; 
 
    try { 
      servlet = (Servlet) myClass.newInstanc
      servlet.service((ServletRequest) reques
       (ServletResponse) response); 
    } 
    catch (Exception e) { 
      System.out.println(e.toString()); 
    } 
    catch (Throwable e) { 
      System.out.println(e.toString()); 
    } 
 
  } 
}
```

此处有一个严重的安全问题：

```
    try {
       servlet = (Servlet) myClass.newInstanc
       servlet.service((ServletRequest) request, (ServletResponse) response); 
    }
```

将reques, response参数传入servlet.service时，用到了向上转型。但是若servlet的使用者知道request, response的实际类型
就可以向下转型，使用request, response中的方法,而这两个类中的部分方法是我们不希望用户使用的，因此此处要使用一个门面模式。

```
Listing 2.7 shows an incomplete RequestFacade class. 
Listing 2.7: The RequestFacade class  
 
package ex02.pyrmont; 
public class RequestFacade implements ServletRequest { 
  private ServleLRequest request = null; 
 
  public RequestFacade(Request request) { 
    this.request = request; 
  } 
 
  /* implementation of the ServletRequest*/ 
  public Object getAttribute(String attribute) { 
    return request.getAttribute(attribute); 
  }
 
  public Enumeration getAttributeNames() { 
    return request.getAttributeNames(); 
  } 
 
  ... 
} 
```

将上述代码替换为：

```
Servlet servlet = null;
RequestFacade requestFacade = new RequestFacade(request);
ResponseFacade responseFacade = new ResponseFacade(response); 
try{
    servlet = new (Servlet) myClass.newInstance();
    servlet.service((ServletRequest) requestFacade, (ServletResponse) responseFacade); 
}
```

done.
