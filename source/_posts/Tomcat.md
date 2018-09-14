---
title: Tomcat——持续更新
date: 2018-08-29 23:25:48
tags: tomcat
categories: Java
---
## Tomcat—持续更新
### 组成
![](https://lc-mhke0kuv.cn-n1.lcfile.com/9ccc3ed9de0df39faa1e.jpeg)
Server：指的就是整个 Tomcat 服 务器，包含多组服务，负责**管理和 启动各个 Service**，同时监听 8005 端口发过来的 shutdown 命令，用 于关闭整个容器 ；
Service：Tomcat 封装的、对外提 供完整的、基于组件的 web 服务， 包含 Connectors、Container 两个 核心组件，以及多个功能组件，各 个 Service 之间是独立的，但是共享 同一 JVM 的资源 ；
**Connector**：Tomcat 与外部世界的连接器，监听固定端口接收外部请求，传递给 Container，并 将 Container 处理的结果返回给外部；
**Container**：Catalina，Servlet 容器，内部有多层容器组成，用于管理 Servlet 生命周期，调用 servlet 相关方法。
**Loader**：封装了 Java ClassLoader，用于 Container 加载类文件； 
Realm：Tomcat 中为 web 应用程序提供访问认证和角色管理的机制；
JMX：Java SE 中定义技术规范，是一个为应用程序、设备、系统等植入管理功能的框架，通过 JMX 可以**远程监控 Tomcat 的运行状态**；
**Jasper**：Tomcat 的 Jsp 解析引擎，用于将 Jsp 转换成 Java 文件，并编译成 class 文件。 **Session**：负责管理和创建 session，以及 Session 的持久化(可自定义)，支持 session 的集
群。
**Pipeline**：在容器中充当管道的作用，管道中可以设置各种 valve(阀门)，请求和响应在经由管 道中各个阀门处理，提供了一种灵活可配置的处理请求和响应的机制。
**Naming**：命名服务，JNDI， Java 命名和目录接口，是一组在 Java 应用中访问命名和目录服务的 API。命名服务将名称和对象联系起来，使得我们可以用名称访问对象，目录服务也是一种命名 服务，对象不但有名称，还有属性。Tomcat 中可以使用 JNDI 定义数据源、配置信息，用于开发 与部署的分离。


### **Container**
 ![](https://lc-mhke0kuv.cn-n1.lcfile.com/1a2613edf5779c7bf184.jpeg)

 - Engine：Servlet 的顶层容器，包含一 个或多个 Host 子容器；
 - Host：虚拟主机，**负责 web 应用的部署和 Context 的创建**；
 - **Context**：Web 应用上下文，包含多个 Wrapper，负责 web 配置的解析、管 理所有的 Web 资源；
 -  Wrapper：最底层的容器，是**对 Servlet 的封装**，负责 Servlet 实例的创 建、执行和销毁。



### 生命周期
- Tomcat 为了方便管理组件和容器的生命周期，定义了从创建、启动、到停止、销毁共 12 中状态。组件和容器只需实现相应的生命周期 方法即可完成各生命周期内的操作(initInternal、startInternal、stopInternal、 destroyInternal)
![](https://lc-mhke0kuv.cn-n1.lcfile.com/1ea5e727c9ad4ca37e05.jpeg)
- Tomcat 的生命周期管理引入了**事件机制**，在组件或容器的生命周期状态发生变化时会通 知事件监听器，**监听器**通过判断事件的类型来进行相应的操作。事件监听器可以在 server.xml 文件中进行配置，Tomcat 各类容器的配置过程就是通过添加 listener 的方式来进行的。
  - EngineConfig：主要打印启动和停止日志
  - **HostConfig**：主要处理部署应用，解析应用 META-INF/context.xml 并创建应用的 Context   
  - **ContextConfig**：主要解析并合并 web.xml，扫描应用的各类 web 资源 (filter、servlet、listener)
### 启动过程
![](https://lc-mhke0kuv.cn-n1.lcfile.com/94989563f76b0c2b6b19.jpeg)
所有容器都是继承自 ContainerBase，基类中封装了容器中的重复工作，负责启动容器相关的组 件 Loader、Logger、Manager、Cluster、Pipeline，启动子容器(线程池并发启动子容器，通过 线程池 submit 多个线程，调用后返回 Future 对象，线程内部启动子容器，接着调用 Future 对象 的 get 方法来等待执行结果)。

### Web应用部署配置
#### server.xml
**catalina.home**：安装目录; **catalina.base**：工作目录,默认值 user.dir
-  修改默认端口(connector的port属性)，添加虚拟主机
- 部署外部应用
```
	  <!--appBase默认\$catalina.base/webapps/-->
      <Host name="localhost"  appBase="webapps"
            unpackWARs="true" autoDeploy="true">
    		<Context path="应用名" docBase="路径">
      </Host>
      或conf/catalana/localhost下创建   应用名.xml   文件
      <Context docBase="路径">
```
1. HostConfig 监听了 StandardHost 容器的事件，在 start 方法中解析上述配置文件(HostConfig会定期检查context.xml)：
- 扫描 appbase 路径下的所有文件夹和 war 包，ContextConfig解析各个应用的 META-INF/context.xml，并 创建 StandardContext，并将 Context 加入到 Host 的子容器中。
- 解析$catalina.base/EngineName/HostName/下的所有 Context 配置，找到相应 web 应 用的位置，ContextConfig解析各个应用的 META-INF/context.xml，并创建 StandardContext，并将 Context 加入到 Host 的子容器中。 

2. ContextConfig 解析 context.xml 顺序：
- 先解析全局的配置 config/context.xml
- 然后解析 Host 的默认配置 EngineName/HostName/context.xml.default
- 最后解析应用的 META-INF/context.xml

3. ContextConfig 解析 web.xml 顺序：

- 先解析全局的配置 config/web.xml
- 然后解析 Host 的默认配置 EngineName/HostName/web.xml.default 接着解析应用的 MEB-INF/web.xml
- 扫描应用 WEB-INF/lib/下的 jar 文件，解析其中的 META-INF/web-fragment.xml 最后合并 xml 封装成 WebXml，并设置 Context

### 请求处理过程
![](https://lc-mhke0kuv.cn-n1.lcfile.com/36a5730697cd0e18a7f5.png)

[原文链接](https://juejin.im/post/58eb5fdda0bb9f00692a78fc)