## Java Web 演变

### 新建 Web 项目

#### Servlet + JSP

- 手动管理依赖包（WEB-INF/lib），加 jstl.jar，servlet-api.jar 等 web 核心包
- 编写 Servlet 
- 配置 web.xml，增加 servlet 映射 url
- 其他 JSP
- 部署到 Tomcat、Resin 等 Web 容器

#### SpringMVC

- Maven管理依赖包（pom.xml），配置文件加入 web 核心包
- 编写 Controller
- 配置 web.xml，声明 SpringMVC 的 DispatcherServlet
- 其他 JSP（前后端代码逐渐分离，前端页面通过 Ajax 与 Controller 接口交互）
- 部署到 Tomcat、Resin 等 Web 容器

#### [SpringBoot](../Spring/springboot.md)

- 按照 IDEA 向导构建工程
- 编写 Controller
- jar 包方式启动（使用内置 Tomcat 运行）



