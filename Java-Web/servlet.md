Listener 是 Servlet 提供的扩展点，一般用于对特定对象的生命周期和特定事件进行响应处理。

> ServletContextListener 是 ServletContext 的监听者，如果 ServletContext 发生变化，如服务器启动时 ServletContext 被创建，服务器关闭时 ServletContext 将要被销毁。
>
> 在JSP文件中，application 是 ServletContext 的实例，由 JSP 容器默认创建。Servlet 中调用 getServletContext()方法得到 ServletContext 的实例。

![](mix/servlet-listener.png)



## 巨人肩膀

[Servlet 工作原理解析](https://www.ibm.com/developerworks/cn/java/j-lo-servlet/index.html)，IBM，2011

