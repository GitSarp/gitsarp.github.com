---
title: Servlet和Sping过滤器
date: 2018-07-09 21:37:34
tags: Spring
categories: Spring
---
## Servlet容器与Tomcat
Tomcat 的容器等级中，Context 容器是直接管理 Servlet 在容器中的包装类 Wrapper，所以 Context 容器如何运行将直接影响 Servlet 的工作方式（一个 Context容器对应一个 Web 应用（工程），也就是Servlet 运行时的 Servlet 容器）。
![Tomcat 的容器等级](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/image002.jpg)

- Tomcat 的配置文件中配置应用
```
<Context path="/projectOne " docBase="D:\projects\projectOne"
reloadable="true" />
```
- tomcat添加一个应用
```
Tomcat tomcat = getTomcatInstance(); 
File appDir = new File(getBuildDirectory(), "webapps/examples"); 
tomcat.addWebapp(null, "/examples", appDir.getAbsolutePath()); 
tomcat.start(); 
ByteChunk res = getUrl("http://localhost:" + getPort() + 
              "/examples/servlets/servlet/HelloWorldExample"); 
assertTrue(res.toString().indexOf("<h1>Hello World!</h1>") > 0);
```
```
//Tomcat.addWebapp
public Context addWebapp(Host host, String url, String path) { 
       silence(url); 
       Context ctx = new StandardContext();//创建一个容器 
       ctx.setPath( url );//访问地址，和配置文件中一致 
       ctx.setDocBase(path);//实际地址，和配置文件中一致 
       if (defaultRealm == null) { 
           initSimpleAuth(); 
       } 
       ctx.setRealm(defaultRealm); 
       ctx.addLifecycleListener(new DefaultWebXmlListener()); 
       ContextConfig ctxCfg = new ContextConfig();//这个类将会负责整个 Web 应用配置的解析工作 
       ctx.addLifecycleListener(ctxCfg); 
       ctxCfg.setDefaultWebXml("org/apache/catalin/startup/NO_DEFAULT_XML"); 
       if (host == null) { 
           getHost().addChild(ctx); 
       } else { 
           host.addChild(ctx); 
       } 
       return ctx; 
}
```
- Tomcat 的启动逻辑是基于观察者模式设计的，所有的容器都会继承 Lifecycle 接口，它管理者容器的整个生命周期，所有容器的的修改和状态的改变都会由它去通知已经注册的观察者
[原文链接](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/index.html)
## Servlet

- servlet容器负责加载和实例化Servlet，在容器启动时根据设置决定是在启动时初始化（loadOnStartup大于等于0，值越小优先级越高），还是延迟初始化直到第一次请求前
- init()方法执行一次性的动作，可以通过ServletConfig配置对象，获取初始化参数，访问ServletContext上下文环境
- 配置：1.通过@WebServlet的initParams属性来指定。2.通过在web.xml文件中配置
- Servlet默认是线程不安全的，单例多线程，一个容器中只有每个servlet一个实例。StandardWrapper负责Servlet的创建，其中SingleThreadModule模式下创建的实例数不能超过20个，也就是同时只能支持20个线程访问这个Serlvet。
- Servlet多线程机制背后有一个线程池在支持，线程池在初始化初期就创建了一定数量的线程对象，通过提高对这些对象的利用率，避免高频率地创建对象，从而达到提高程序的效率的目的。
## Spring过滤器
用来实现一些特殊的功能。例如URL级别的权限访问控制、过滤敏感词汇、压缩响应信息等。常见的有CharacterEncodingFilter、InvilidCharacterFilter(防止脚本攻击)等。
### 具体步骤
1. 编写Filter
```
public class FilterTest implements Filter{
    public void destroy() {
        System.out.println("----Filter销毁----");
    }
 
    public void doFilter(ServletRequest request, ServletResponse response,FilterChain filterChain) throws IOException, ServletException {
        // 对request、response进行一些预处理
        request.setCharacterEncoding("UTF-8");
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html;charset=UTF-8");
        System.out.println("----调用service之前执行一段代码----");
        filterChain.doFilter(request, response); // 执行目标资源，放行
        System.out.println("----调用service之后执行一段代码----");
    }
 
    public void init(FilterConfig arg0) throws ServletException {
        System.out.println("----Filter初始化----");
    }
}
```
- FilterConfig(init()方法的入参)获取配置的初始化参数
   ```
　　String getFilterName()：得到filter的名称。
　　String getInitParameter(String name)： 返回在部署描述中指定名称的初始化参数的值。如果不存在返回null.
　　Enumeration getInitParameterNames()：返回过滤器的所有初始化参数的名字的枚举集合。
　　public ServletContext getServletContext()：返回Servlet上下文对象的引用。
   ```
- FilterChain
web服务器将Filter组合起来形成为一个Filter链。根据Filter在web.xml文件中的注册顺序，决定先调用哪个Filter，当第一个Filter的doFilter方法被调用时，web服务器会创建一个代表Filter链的FilterChain对象传递给该方法。在doFilter方法中，如果调用了FilterChain对象的doFilter方法，则web服务器会检查FilterChain对象中是否还有filter，如果有，则调用第2个filter，如果没有，则调用目标资源。
2. 注册
```
<web-app ...>
  ...
  <!--配置过滤器-->
  <filter>
      <filter-name>FilterTest</filter-name>
      <filter-class>com.freaxjj.filter.FilterTest</filter-class>
      [<init-param>
        <description>配置FilterTest过滤器的初始化参数</description>
        <param-name>like</param-name>
        <param-value>java</param-value>
      </init-param>]
  </filter>
  <!--映射过滤器-->
  <filter-mapping>
      <filter-name>FilterTest</filter-name>
      <url-pattern>/*</url-pattern>
      [<servlet-name>、<dispatcher>]
  </filter-mapping>
</web-app>
```
- <servlet-name>指定过滤器所拦截的Servlet名称。
- <dispatcher>指定过滤器所拦截的资源被 Servlet 容器调用的方式。默认子元素REQUEST。用户可以设置多个子元素用来指定 Filter 对资源的多种调用方式进行拦截
  -  REQUEST：当用户直接访问页面时，Web容器将会调用过滤器。如果目标资源是通过RequestDispatcher的include()或forward()方法访问时，那么该过滤器就不会被调用。
  - INCLUDE：如果目标资源是通过RequestDispatcher的include()方法访问时，那么该过滤器将被调用。除此之外，该过滤器不会被调用。
  - FORWARD：如果目标资源是通过RequestDispatcher的forward()方法访问时，那么该过滤器将被调用，除此之外，该过滤器不会被调用。
  - ERROR：如果目标资源是通过声明式异常处理机制调用时，那么该过滤器将被调用。除此之外，过滤器不会被调用。
### 实践
```
public class SelfDefineInvalidCharacterFilter implements Filter{
 
    public void destroy() {
        
    }
 
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        String parameterName = null;
        String parameterValue = null;
        // 获取请求的参数
        @SuppressWarnings("unchecked")
        Enumeration<String> allParameter = request.getParameterNames();
        while(allParameter.hasMoreElements()){
            parameterName = allParameter.nextElement();
            parameterValue = request.getParameter(parameterName);
            if(null != parameterValue){
                for(String str : invalidCharacter){
                    if (StringUtils.containsIgnoreCase(parameterValue, str)){
                        request.setAttribute("errorMessage", "非法字符：" + str);
                        RequestDispatcher requestDispatcher = request.getRequestDispatcher("/error.jsp");
                        requestDispatcher.forward(request, response);
                        return;
                    }
                }
            }
        }
        filterChain.doFilter(request, response); // 执行目标资源，放行
    }
 
    public void init(FilterConfig filterConfig) throws ServletException {
        
    }
    // 需要过滤的非法字符
    private static String[] invalidCharacter = new String[]{
        "script","select","insert","document","window","function",
        "delete","update","prompt","alert","create","alter",
        "drop","iframe","link","where","replace","function","onabort",
        "onactivate","onafterprint","onafterupdate","onbeforeactivate",
        "onbeforecopy","onbeforecut","onbeforedeactivateonfocus",
        "onkeydown","onkeypress","onkeyup","onload",
        "expression","applet","layer","ilayeditfocus","onbeforepaste",
        "onbeforeprint","onbeforeunload","onbeforeupdate",
        "onblur","onbounce","oncellchange","oncontextmenu",
        "oncontrolselect","oncopy","oncut","ondataavailable",
        "ondatasetchanged","ondatasetcomplete","ondeactivate",
        "ondrag","ondrop","onerror","onfilterchange","onfinish","onhelp",
        "onlayoutcomplete","onlosecapture","onmouse","ote",
        "onpropertychange","onreadystatechange","onreset","onresize",
        "onresizeend","onresizestart","onrow","onscroll",
        "onselect","onstaronsubmit","onunload","IMgsrc","infarction"
    }; 
}

```
### 生命周期
Filter的创建和销毁由web服务器负责。 web应用程序启动时，web服务器将创建Filter的实例对象，并调用其init方法。filter对象只会创建一次。init方法和destroy方法在Filter的生命周期中仅执行一次。在destroy方法中，可以释放过滤器使用的资源。

[原文链接](https://blog.csdn.net/reggergdsg/article/details/52821502)