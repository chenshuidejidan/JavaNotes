# Maven
- 依赖管理：
- 项目的一键构建：编译、测试、运行、打包、安装、部署 整个过程都交给maven来进行管理。

## 1. 基础知识
- 环境变量中添加 MAVEN_HOME
- Path 中添加 %MAVEN_HOME%\bin
- 默认本地仓库位置：`Default: ${user.home}/.m2/repository`，本地仓库没有时去中央仓库寻找
![wDcPtP.png](https://s1.ax1x.com/2020/09/14/wDcPtP.png)

- 修改本地仓库：`<localRepository>D:\maven_repository</localRepository>`
- maven项目结构：核心代码、配置文件、测试代码、测试配置文件
- 标准目录结构
  - `src/main/java` : 核心代码
  - `src/main/resources` : 配置文件
  - `src/test/java` : 测试代码
  - `src/test/resources` : 测试配置文件
  - `src/main/webapp` : 页面资源，js，css，图片等
## 2. 常用命令
- `mvn compile` : 编译src/main 放置在target中
- `mvn clean` : 清除本地编译信息，删除target目录
- `mvn test` : 编译**src/test和src/main**
- `mvn package` : 打包项目成.var(根据pom中`<packaging>war</packaging>`)，放置在target目录中
- `mvn install` : 将改项目编译，打包，并安装到本地仓库
- 整个项目构建过程：**清除项目编译信息(clean)、编译(compile)、测试(test)、打包(package)、安装(insatall)、发布(deploy)** 均有对应命令
- 生命周期：清理生命周期(clean)、**默认生命周期(compile->deploy)**、站点生命周期

## 3. maven 概念模型
![wDfGwQ.png](https://s1.ax1x.com/2020/09/14/wDfGwQ.png)

## 4. maven 的一些配置

- IDEA maven配置：Maven->Runner->VM Options:`-DarchetypeCatalog=internal`
- maven换源：

~~~xml
  <mirrors>
    <mirror>
    <id>aliyunmaven</id>
    <mirrorOf>central</mirrorOf>
    <name>aliyun maven</name>
    <url>https://maven.aliyun.com/repository/public </url>
    </mirror>
  </mirrors>
~~~
- 坑：`java: 程序包org.springframework.boot不存在` 新版IDEA需要在Setting里将 delegate IDE build/run actions to Maven勾选上即可
- 使用骨架创建web工程：Maven->Create from archtype->maven-archetype-webapp
- **jdk1.8以上需要运行tomcat7以上版本！！！！(默认是6)** ， 使用 `mvn tomcat7:run` 运行
~~~xml
        <plugin>
          <groupId>org.apache.tomcat.maven</groupId>
          <artifactId>tomcat7-maven-plugin</artifactId>
          <version>2.2</version>
          <configuration>
            <uriEncoding>UTF-8</uriEncoding>
            <path>/</path>
            <port>8080</port>
          </configuration>
        </plugin>
~~~
- **jar包冲突**：tomcat中有servlet和servlet.jsp包，所以冲突，解决办法：设置scope作用范围为 `provided`

~~~xml
  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet</groupId>
      <artifactId>servlet-api</artifactId>
      <version>2.5</version>
      <scope>provided</scope>
    </dependency>
    <dependency>
      <groupId>javax.servlet.jsp</groupId>
      <artifactId>jsp-api</artifactId>
      <version>2.0</version>
      <scope>provided</scope>
    </dependency>
  </dependencies>
~~~
- **最后记得配置web.xml中的servlet映射：**

~~~xml
  <servlet>
    <servlet-name>MyServlet</servlet-name>
    <servlet-class>com.jj.servlet.MyServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>MyServlet</servlet-name>
    <url-pattern>/MyServlet</url-pattern>
  </servlet-mapping>
~~~

## 5. 常见的scope作用域例子

|依赖范围|编译classpath有效|测试classpath有效|运行时classpath有效|例子|
|---|:---:|:---:|:---:|---|
|compile|Y|Y|Y|spring-core|
|test|-|Y|-|Junit|
|**provided**|Y|Y|-|**servlet-api**|
|**runtime**|-|Y|Y|**JDBC驱动**|
|system|Y|Y|-|本地的，Maven仓库之外的类库<br>显示提供本地jar路径，一般不使用|


# Spring
解耦，简化开发；AOP编程，实现传统OOP不易实现的功能；声明式事务的支持；方便测试；方便对各种优秀框架的支持(Structs,Hibernate,Hessian等)；降低JavaEE API的使用难度，如JDBC、JavaMail等；源码是经典的学习范例

## 1. Spring 简介入门

### 1.1 Spring概览
Spring体系结构
![Spring体系结构](https://s1.ax1x.com/2020/09/14/wDMp8S.gif)

Spring开发步骤
![wDMqRU.png](https://s1.ax1x.com/2020/09/14/wDMqRU.png)
1. 导入Spring开发的基本包坐标
2. 编写Dao接口和实现类
3. 创建Spring核心配置文件
4. 在Spring配置文件中配置UserDaoImpl
5. 使用Spring的API获得Bean实例
### 1.2 QuickStart
