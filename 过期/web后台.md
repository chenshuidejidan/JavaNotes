常见的java web服务器软件     
- webLogic : oracle的大型 JavaEE 服务器，支持所有的 JavaEE 规范
- webSphere : IBM，支持所有JavaEE 规范  
- JBOSS : JBOSS，支持所有JavaEE 规范    
- Tomcat ： Apache 基金组织，中小型的JavaEE 服务器，仅支持少量的JavaEE 规范servlet/jsp，开源的


# Tomcat
## 1. 目录
- bin ： 可执行文件
- conf ： 配置文件
- lib ： 依赖jar包
- logs ： 日志文件
- temp ：临时文件 
- webapps : 存放web项目
- work ： 存放tomcat运行时的数据：**jsp被访问后生成对应的server文件和.class文件**      

动态项目：webapps/项目根目录/WEB-INF/ 下
- web.xml ： web项目的核心配置文件
- classes ： 放置字节码文件的目录  
- lib : 放置依赖的 jar 包
## 2. 配置  
- 运行Tomcat需要JDK的支持【Tomcat会通过JAVA_HOME找到所需要的JDK】
### 2.1 端口
- 查看电脑端口占用情况`netstat -ano` 可以查到进程对应PID        
- 修改TomCat默认端口号：server.xml，一般修改为80，http协议默认端口，访问时可以不输入端口号  
### 2.2 虚拟主机
- 开发多个网站时，有多个域名。如果不配置虚拟主机，一个Tomcat服务器运行一个网站，就需要多台电脑才能把这些网站运行起来
- 在tomcat的server.xml文件中添加主机名

~~~xml
<Host name="xxxxxxx" appBase="D:\hello1">
  <Context path="/hello1" docBase="D:\hello1"/>
</Host>
~~~

## 3. 部署项目的方法
1. **直接**把项目文件夹复制到 webapps 目录下
2. 将项目**打包为.war**，复制到webapps会自动解压缩，删除.war自动删除项目
3. 配置虚拟目录：conf/server.xml，**重启服务器才会生效**   
~~~xml
<Host>
...
<!-- docBase：项目存放路径  path：虚拟目录 -->
<Context docBase="D:\hello" path="/hi" />
</Host>
~~~

4. 配置虚拟目录： conf\catalina\localhost\xxx.xml **创建任意xml**    
虚拟目录就是 xml 的文件名
~~~xml
<!-- 不配置path，则为/，可以直接访问 -->
<Context docBase="D:\hello" />
~~~

# Servlet           
- JavaWeb的三大组件：Servlet、Filter、Listener
- 运行在服务器端的小程序，其实就是一个接口，定义了 java 类被浏览器访问到(tomcat识别)的规则  
- 自定义类实现Servlet接口，就可以被tomcat识别

## 1. QuickStart
- 创建JavaEE项目
- 定义一个类实现Servlet接口的抽象方法
- web.xml中配置Servlet
~~~xml
    <!--配置Servlet-->
    <servlet>
        <servlet-name>demo1</servlet-name>
        <servlet-class>cn.servlet.ServletDemo1</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>demo1</servlet-name>
        <!-- 配置虚拟路径 -->
        <url-pattern>/demo1</url-pattern>
    </servlet-mapping>
~~~

## 2. Tomcat 执行原理
1. 浏览器访问项目的资源路径
2. tomcat检索web.xml，寻找是否有对应的url-pattern
3. tomcat根据url-pattern找到servlet-name，再找到对应的servlet-class全类名
4. tomcat利用反射机制将servlet-class字节码文件加载进内存，创建对象
5. tomcat调用其方法

## 3. 方法
- `void init(ServletConfig servletConfig)`  初始化，初次访问Servlet时，只执行一次
- `ServletConfig getServletConfig()` 获取配置文件 
- `void service(ServletRequest servletRequest, ServletResponse servletResponse)`  提供服务，每次Servlet被访问都会执行
- `String getServletInfo()` 获取Servlet信息：作者版本等。。。  
- `void destroy()`  Servlet销毁(服务器正常关闭)时，只执行一次

## 4. Servlet 生命周期  
**1. 被创建：**     
执行init方法，只执行一次。默认情况下，第一次被访问时，Servlet 被实例化并调用init初始化，可以修改 web.xml 指定 Servlet 的创建时机。          
Servelet的init方法只执行一次，说明一个Servlet在内存中只存在一个对象，是单例的，多用户同时访问时，可能存在线程安全问题，所以尽量不要在Servlet中定义成员变量，即使定义了成员变量，也不要修改值    
~~~xml
<servlet>
...
<!-- 为负数时标识第一次被访问时创建，为0和正数则在服务器启动时创建 -->
<!-- 正数的值越小，该servlet的优先级越高，应用启动时就越先加载 -->
<!-- 当值相同时，容器就会自己选择顺序来加载 -->
<!-- 所以取值0，1，2，3，4，5代表的是优先级，而非启动延迟时间 -->
<load-on-startup>5</load-on-start>
</servlet>
~~~
**2. 提供服务：**           
执行service方法，每次访问 Servlet时，Service 方法都会被调用一次         

**3. 被销毁：**     
执行destroy方法，服务器**正常关闭**时执行，一般用于释放资源，执行destory方法后Servlet被销毁       

## 5. Servlet3.0          
- 创建JavaEE项目时，可以不创建 web.xml   
**注解配置：**      
Webservlet 注解
~~~java
public @interface WebServlet {
    String name() default "";
    String[] value() default {};
    String[] urlPatterns() default {};
    int loadOnStartup() default -1;
    WebInitParam[] initParams() default {};
    boolean asyncSupported() default false;
    String smallIcon() default "";
    String largeIcon() default "";
    String description() default "";
    String displayName() default "";
}
~~~

使用注解配置：
~~~java
@WebServlet(urlPatterns="/demo2")
public class ServletDemo2 implements Servlet {
    ...
}
~~~
因为value是最重要的属性，就是urlPatterns，所以可以直接写value="/demo2"，如果只有一个属性时value也可以省略：
~~~java
@WebServlet("/demo2")
public class ServletDemo2 implements Servlet {
    ...
}
~~~

- `urlPatterns()` 可以传入多个参数，浏览器从多个虚拟路径访问该 Servet，路径还可以是**多级结构** xxx/xxx，也可以使用**通配符** xxx/* ， */xxx，通配符的优先级低，还可以使用 **\*.do** 的形式，表示只匹配以 .do 结尾  

## 6. IDEA和tomcat相关配置        
- IDEA为每个tomcat部署的项目单独建立了一份配置文件          
~~~
Using CATALINA_BASE:   "C:\Users\lenovo\AppData\Local\JetBrains\IntelliJIdea2020.1\tomcat\_Java_learn"
~~~
![dF75tK.png](https://s1.ax1x.com/2020/08/15/dF75tK.png)
- tomcat真正访问的是tomcat部署的web项目，对应的是工作空间项目的web目录下资源 以及 src下的文件对应的class文件        
- WEB-INF 下的资源不能被浏览器直接访问    

## 7. Servlet 体系结构
- Servlet接口的两个实现类(抽象类)：`GenericServlet` 和 `HttpServlet` (接口适配器)
- `GenericServlet` : 只有service方法是抽象的，其他方法都进行了默认的空实现
- `HttpServlet` ：对http协议的封装，继承并重写 `doGet()`，`doPost()` 

  
## 8. Request 
<center>

![请求报文2.png](https://s1.ax1x.com/2020/08/15/dkIcKs.png)
</center>

### 8.1 请求行
- 请求行格式：`请求方式 url 协议/版本` 如 `GET /demo1/login.html?name=zhangsan HTTP/1.1`

**请求方式：**          
- GET：参数在请求行中，在url后，长度有限制，不太安全
- POST：参数在请求体中，长度没有限制，相对安全

### 8.2 请求头   
- 请求头格式：`头部字段：值`       
- 浏览器告诉服务器的一些信息            


**常见的请求头：**      
- **User-Agent：** 产生请求的浏览器类型，可以在服务端获取据浏览器类型版本来解决兼容性问题           
- **Host：** 请求的主机名，允许多个域名同处一个IP地址，即虚拟主机       
- **Accept：** 客户端可识别的内容类型列表（如：Accept: text/html,image/*    【浏览器告诉服务器，它支持的数据类型】）      
- **Reffer：** 告诉服务器，当前请求是从哪个链接发出。。防止盗取链接，做统计工作         
- **Connection：** keep-alive
- **Cookie：** 浏览器告诉服务器，带来的Cookie


### 8.3 请求空行
- 回车换行，分割请求头和请求体

### 8.4 请求体(正文)
- POST 方式

### 8.5 Request 对象         
- Request和Response对象都是**由tomcat创建**的       
- request对象中封装了请求消息的数据，tomcat将**request和response对象传递给service方法**，并且调用service方法
- 我们可以通过request对象获取请求消息数据，通过response设置响应消息数据
- 服务器给浏览器做出响应前就会从response对象中提取响应消息数据

**Request 继承体系**        
- ServletRequest 接口
- HttpServletRequest 接口 继承了 ServletRequest
- org.apache.catalina.connector.RequestFacade 类(tomcat) 实现了 HttpServletRequest 接口     

### 8.6 Request 对象的方法      
#### 8.6.1 获取请求行               
`GET /tomcat/demo1?name=zhangsan HTTP/1.1`
- `String getMethod()` ： 获取请求方式 (GET)
- `String getContextPath()` : **获取虚拟目录 (/tomcat)**
- `String getServletPath()` ： 获取Servlet路径 (/demo1)
- `String getRequestURI()` ： **获取请求URI (/tomcat/demo1)**
- `String getRequestURL()` ： 获取请求URL (http://localhost/tomcat/demo1)
- `String getQueryString()` ： 获取get的请求参数 (name=zhangsan)
- `String getProtocol()` ： 获取协议        
- `String getRemoteAddr()` ： 获取客户机的IP地址        

#### 8.6.2 获取请求头           
- `Enumeration<String> getHeaderNames()` ： 获取所有的请求头字段名(迭代器)
- `String getHeader(String name)` ： **通过请求头的字段名获取值**          
~~~java
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取请求头字段名的迭代器
        Enumeration<String> headerNames = request.getHeaderNames();
        while(headerNames.hasMoreElements()){
            String name = headerNames.nextElement();
            //根据请求头的字段名获取值
            String value = request.getHeader(name);
            System.out.println(name+"------"+value);
        }
    }
~~~

#### 8.6.3 获取请求体           
只有POST请求才有        

首先获得流对象，再从流中获取数据                  
- `BufferedReader getReader()` ： 获取字符输入流          
- `ServletInputStream getInputStream()` ： 获取字节输入流       

~~~java
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        BufferedReader reader = request.getReader();
        System.out.println(reader.readLine());
    }
~~~

    
#### 8.6.4 获取请求参数通用方式（post和get通用）      
- `String getParameter(String name)` ： **根据参数名称获取参数值**           
- `String[] getParameterValues(String name)` ： **根据参数名称获取参数值的数组（复选框）**              
- `Enumeration<String> getParameternames()` ： **获取所有请求参数名称的数组**             
- `Map<String, String[]> getParameterMap()` ： **获取所有参数的键值对集合**    
- post方式，控制台输出中文乱码的问题： 设置流的编码`request.setCharacterEncoding("utf-8")`       
~~~java
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        request.setCharacterEncoding("utf-8");
        Enumeration<String> parameterNames = request.getParameterNames();
        while (parameterNames.hasMoreElements()) {
            System.out.println(request.getParameter(parameterNames.nextElement()));
        }
    }
~~~          

#### 8.6.5 请求转发：在服务器内部的资源跳转方式          
`RequestDispatcher getRequestDispatcher(String path)` : 通过request对象获取请求转发器对象           
`requestDispatcher.forward(ServletRequest request, ServletResponse response)` : 使用RequestDispatcher对象进行转发   
**请求转发的特点：** 浏览器地址栏路径不发生变化，只能转发到当前服务器的内部资源，转发是一次请求

#### 8.6.6 共享数据              
**域对象：** 有作用范围的对象，可以在范围内共享数据             
**request域：** 代表一次请求的范围，一般用于请求转发的多个资源中共享数据        
`setAttribute(String name, Object obj)` ： 存储数据         
`Object getAttribute(String name)` ： 通过键获取值      
`void removeAttribute(String name)` : 通过键删除键值对
~~~java
@WebServlet("/demo5")
public class Demo5RequestDispatcher1 extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("进入到了demo5");
        //存储数据
        req.setAttribute("key1","value1");
        //请求转发
        req.getRequestDispatcher("/demo6").forward(req, resp);
    }
}

@WebServlet("/demo6")
public class Demo6RequestDispatcher2 extends HttpServlet {
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("进入到了demo6");
        System.out.println(request.getAttribute("key1"));
    }
}

~~~
#### 8.6.7 获取ServeletContext
- `ServletContext getServletContext()`
- 也可以直接通过HttpServlet获取：`this.getServletContext()`   
- ServletContext 代表整个web应用，可以和程序的容器(服务器)通信  


**SrevletContext功能**          
- 获取MIME类型 `getMimeType()`：MIME类型是互联网通信中定义的一种文件数据类型，格式：大类型/小类型，如：text/html，image/jpg
- ServletContext是域对象，**可以共享所有请求数据**(与request域区分，ServletContext域无需转发)
- 获取文件的真实路径(服务器路径) `getRealPath()`：          
<center>

![realPath.png](https://s1.ax1x.com/2020/08/17/dmgii8.png)
</center>   

~~~java
@WebServlet("/demo9")
public class Demo9ServletContext extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取ServletContext
        ServletContext servletContext = request.getServletContext();
        ServletContext servletContext1 = this.getServletContext();

        System.out.println(servletContext==servletContext1);   //true

        //获取MIME
        String filename = "a.jpg";
        System.out.println(servletContext.getMimeType(filename));   // image/jpg

        //servletContext共享域，所有用户都可以获得该共享数据
        servletContext.setAttribute("name", "aaa");

        //获取真实路径
        System.out.println(servletContext.getRealPath("/"));    //根路径是web的路径
        System.out.println(servletContext.getRealPath("/b.txt"));  //b 在web下
        System.out.println(servletContext.getRealPath("/WEB-INF/c.txt"));  // web/WEB-INF 下
        System.out.println(servletContext.getRealPath("/WEB-INF/classes/a.txt")); // src目录下的文件最后会到WEB-INF/classes中
        /*classLoader只能获取src下的路径，不能获取到部署后在web下的路径

          C:\Users\lenovo\IdeaProjects\Java_learn\out\artifacts\tomcat_war_exploded\
          C:\Users\lenovo\IdeaProjects\Java_learn\out\artifacts\tomcat_war_exploded\b.txt
          C:\Users\lenovo\IdeaProjects\Java_learn\out\artifacts\tomcat_war_exploded\WEB-INF\c.txt
          C:\Users\lenovo\IdeaProjects\Java_learn\out\artifacts\tomcat_war_exploded\WEB-INF\classes\a.txt
        */
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doPost(request, response);
    }
}
~~~
#### 8.6.8 登录案例
500错误             
错误描述：java.lang.NoClassDefFoundError: org/springframework/dao/DataAccessException
 
错误原因：lib没在正确的位置，web/WEB-INF/lib


#### 8.6.9 BeanUtils 工具
- org.apache.commons.beanutils.BeanUtils
- BeanUtils 用于封装 JavaBean       
- JavaBean是标准的Java类，要求：**类必须是public**，提供**空参构造**，**成员变量private修饰**，提供**public的setter和getter**            
- **成员变量和属性的区别：** 属性是setter和getter方法截取后的产物，例如 getUsername() -> Username -> username
- BeanUtils 操作的是**属性**    
- `populate(Object bean, Map map)` ： 将 map 集合的键值对信息，按属性封装到对应的 JavaBean 对象
~~~java
        Map<String, String[]> parameterMap = request.getParameterMap();
        User loginUser = new User();
        BeanUtils.populate(loginUser,parameterMap);
~~~
- `getProperty和setProperty`

~~~java
        BeanUtils.setProperty(loginUser, "username","张三");
        String username = BeanUtils.getProperty(loginUser, "username");
~~~

## 9. Response  
<center>

![响应报文.png](https://s1.ax1x.com/2020/08/15/dk5llj.png)
</center>

- HTTP响应也由三个部分组成，分别是：状态行、消息报头、响应正文  
- 在响应中唯一真正的区别在于第一行中用**状态信息**代替了请求信息。状态行（status line）通过提供一个状态码来说明所请求的资源情况 
### 9.1 响应行
响应行格式：`HTTP-Version Status-Code Reason-Phrase CR+LF`             
HTTP-Version表示服务器HTTP协议的版本；Status-Code表示服务器发回的响应状态代码；Reason-Phrase表示状态代码的文本描述。状态代码由三位数字组成，第一个数字定义了响应的类别，且有五种可能取值

#### 9.1.1 响应状态码       

- 1xx：**指示信息**--表示请求已接收客户端消息，但没有接收完成，需要继续处理(客户端继续发送等)

- 2xx：**成功**--表示请求已被成功接收、理解、接受
  - **200 OK 正常处理**        
  - 204 成功处理，但服务器没有新数据返回，显示页面不更新        
  - 206 对服务器进行范围请求，只返回一部分数据
 
- 3xx：**重定向**--要完成请求必须进行更进一步的操作
  - 301 **永久重定向**，请求的资源已分配了新的URI，原URL地址改变了，以后应使用新的URL访问，常用作域名跳转              
  - 302 **临时重定向**，请求的资源临时分配了新的URI中，原URL地址没变，常用作临时跳转如未登录访问用户中心跳转登录页           
  - 303 请求资源路径发生改变，与302功能一样，但明确指出客户端应该采用GET方式来请求新URL          
  - 304 访问的是未更改(未过期)的本地缓存资源        
  - 307 与302相同，但不会把POST请求变成GET      
  - 301与302：用户都可以看到url替换为了一个新的，都根据服务器返回的location进行二次请求，但是301是可以缓存的，在抓取新的内容的同时也将旧的网址替换为了重定向之后的网址，但是302是暂时的，抓取新内容也保留旧的地址

- 4xx：**客户端错误**--请求有语法错误或请求无法实现
  - **400 Bad Request 请求报文语法错误了**      
  - **401 Unauthorized 请求未经授权，需要认证身份**        
  - **403 Forbidden 服务器收到，但是拒绝提供服务**           
  - **404 Not Found 请求的资源不存在**    
  - **405 Method not allowed 请求方式没有对应的doxxx方法**  

- 5xx：**服务器端错误**--服务器未能实现合法的请求        
  - **500 Internal Server Error 服务器内部资源出错了**      
  - **503 服务器正忙，不能处理客户请求，一段时间后恢复服务**  

### 9.2 响应头
- 请求头格式：`头部字段：值`    
#### 9.2.1 常见的响应头
- **Content-Type**: text/html;charset=utf-8 ： content-type 服务器告诉客户端本次响应体数据格式以及编码格式
- **Content-disposition**: in-line ： 服务器告诉客户端以什么格式打开响应体数据(in-line默认值表示在当前页面打开，**attachment表示以附件形式打开响应体如文件下载**时)         
- **location**： 重定向时，指明重定向的资源路径     
- **set-cookie**: 服务器发送cookie给浏览器，浏览器自动保存，下次请求时会在请求头中带上 **cookie**           

### 9.3 响应空行

### 9.4 响应体      
- 传输的数据    

### 9.5 Response 对象的方法 
- 功能：设置响应消息        

#### 9.5.1 设置相应行 
- `HTTP/1.1 200 ok`
- `setStatus(int sc)` ： 设置状态码  
#### 9.5.2 设置响应头
- `setHeader(String name, String value)` ：设置响应头       
- `setContentType(String var1)` : 如`response.setContentType("text/html; charset=UTF-8");`，设置编码格式            


#### 9.5.3 设置响应体   
- 获取输出流：      
`PrintWriter getWriter()` ： 获取字符输出流                 
`ServletOutPutStream getOutPutStream()` ： 获取字节输出流           

- 使用输出流，将数据输出到客户端        
response 获取的输出流使用完**不需要close，不需要flush**，都是自动的
#### 9.5.4 重定向  
`sendRedirect()` ： 简单重定向   
~~~java
//重定向
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //设置状态码
    response.setStatus(302);
    //设置响应头location
    response.setHeader("location","/tomcat/demo2");
}
~~~
~~~java
//简单重定向
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //302和location字段都是固定不变的，只需要设置location的值，原理和上面是相同的
    response.sendRedirect("/tomcat/demo2");
}
~~~

**转发和重定向的区别**          
**转发(forward)**：地址栏路径不变，只能访问当前服务器下的资源，是一次请求，可以用Request域共享信息            
**重定向(redirect)**：地址栏发生变化，可以访问其他站点下的资源，是两次请求，不能用Request域共享信息        

### 9.6 资源路径

#### 9.6.1 相对路径和绝对路径
相对路径：不可以确定唯一资源。 如 ./html/index.html，**./表示当前路径，也可以省略：** html/index.html          
绝对路径：通过绝对路径可以唯一确定资源。如 http://localhost:8080/tomcat/demo2  可以省略为 /tomcat/demo2         

#### 9.6.2 绝对路径的写法   
- 判断资源是给服务器使用还是给客户端使用(访问该资源的请求是从服务器发出的还是客户端发出的)        
- 给客户端浏览器使用：需要加虚拟目录        
- 给服务器用：不加虚拟目录      
- 所以**请求转发不加虚拟目录，而重定向要加虚拟目录**
- **使用 request.getcContextPath 获取虚拟目录**     


~~~java
    //请求转发不用加虚拟目录
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //请求转发
        req.getRequestDispatcher("/demo6").forward(req, resp);
    }

    //重定向必须加虚拟目录
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //重定向
        response.sendRedirect("/tomcat/demo6");
    }

    //但是虚拟目录可能会改变，一改就要改多处，所以最好的写法是把虚拟目录写为动态的
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //使用 request.getcContextPath 获取虚拟目录
        response.sendRedirect(request.getContextPath + "/demo6");
    }
~~~

### 9.7 网页输出的编码问题  
- **设置流的编码和响应的编码格式**
- 设置响应的编码也包含了流编码。。。。设置响应编码的同时也设置了流编码，但是要在获取输出流之前设置        
- `setContentType(String var1)`
~~~java
//字符流输出
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    //设置流的编码，默认为ISO-8859-1
    //response.setCharacterEncoding("utf-8")
    
    //设置响应的编码格式
    response.setContentType("text/html; charset=UTF-8");
    response.getWriter().write("ssss你好");
}
~~~

~~~java
//字节流输出
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    
    //设置响应的编码格式
    response.setContentType("text/html; charset=UTF-8");
    response.getOutputStream().write("ssss你好".getBytes("utf-8"));
}
~~~

### 9.8 图片输出
~~~java
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        int height = 50;
        int width = 100;
        //创建图片对象
        BufferedImage image = new BufferedImage(width,height,BufferedImage.TYPE_INT_RGB);

        //填充背景
        Graphics g = image.getGraphics();   //画笔对象
        g.setColor(Color.pink);
        g.fillRect(0, 0, width, height);

        //画边框
        g.setColor(Color.blue);
        g.drawRect(0, 0, width-1, height-1);

        //写验证码
        g.setColor(Color.black);
        String str = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";

        for (int i=0; i<4; i++) {   //随机生成字母
            char ch = str.charAt(new Random().nextInt(str.length()));
            g.drawString(String.valueOf(ch), width/5*(i+1), height/2);
        }

        //干扰线
        g.setColor(Color.green);
        for (int i = 0; i < 10; i++) {
            Random random = new Random();
            g.drawLine(random.nextInt(width), random.nextInt(height), random.nextInt(width), random.nextInt(height));
        }

        //输出图片
        ImageIO.write(image, "jpg", resp.getOutputStream());

    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }
~~~

### 9.9 下载文件案例    
- html中 通过访问虚拟目录中的servlet进行下载，传递需要下载的文件名    
~~~html
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<a href="/tomcat/downloadServlet?filename=1.jpg">下载图片</a><br>
<a href="/tomcat/downloadServlet?filename=1.avi">下载视频</a>
</body>
</html>
~~~

- servlet中实现下载，利用 **ServletContext.getRealPath()** 获取文件的真实路径       
- 利用输入流将文件读入内存，利用输出流进行输出
- 通过 **ServletContext.getMimeType()** 获取文件类型并在响应头中设置该类型
- 设置 **content-disposition** 为 **attachment** 告知浏览器以附件方式打开       

~~~java
@WebServlet("/downloadServlet")
public class Demo10DownloadServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取文件名
        String filename = request.getParameter("filename");
        //获取真实路径
        String realPath = request.getServletContext().getRealPath("/picture/"+filename);
        //读取文件到内存
        BufferedInputStream inputStream = new BufferedInputStream(new FileInputStream(new File(realPath)));

        //设置response响应头,通过servletContext.getMimeType获取文件mime类型，设置附件打开方式
        response.setContentType(request.getServletContext().getMimeType(filename));
        response.setHeader("content-disposition","attachment;filename="+filename);

        //输出
        BufferedOutputStream outputStream = new BufferedOutputStream(response.getOutputStream());
        byte[] buff = new byte[1024*1024];
        int len = 0;
        while((len=inputStream.read(buff))!=-1){
            outputStream.write(buff,0,len);
        }

        inputStream.close();
        outputStream.close();

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        this.doPost(request, response);
    }
}
~~~

# 会话

## 1. 会话技术
- 会话：一次会话中包含了多次请求和响应。一次会话从浏览器第一次给服务器资源发送请求，会话建立开始，直到有一方断开为止
- 服务器和客户端在一次会话中的多次请求间进行共享数据            
- 客户端会话技术：Cookie
- 服务器端会话技术：Session

## 2. 客户端会话技术Cookie

- 创建Cookie对象，绑定数据：`new Cookie(String name, String name)`      
- 发送Cookie对象，发送数据：`response.addCookie(Cookie cookie)`
- 获取Cookie对象，获取数据：`Cookie[] request.getCookies()`

~~~java
@WebServlet("/Demo1Cookie")
public class Demo1Cookie extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //创建cookie
        Cookie c = new Cookie("msg", "hello");
        //发送cookie
        response.addCookie(c);
    }
    ...
}

@WebServlet("/Demo2Cookie")
public class Demo2Cookie extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //获取cookie
        Cookie[] cookies = request.getCookies();
        if(cookies!=null){
            for(Cookie c:cookies){
                System.out.println("name="+c.getName()+", value="+c.getValue());
            }
        }
    }
    ...
}

~~~

### 2.1 一次发送多个Cookie      
- 创建多个 Cookie 对象，使用 response 调用多次 addCookie() 发送 Cookie   

### 2.2 Cookie保存到硬盘
- 默认情况下，浏览器关闭后，Cookie就被销毁了，保存在内存中            
- 设置持久化存储：`setMaxAge(int seconds)`          
- seconds为正数表示存储到硬盘的存活时间(秒)，负数为默认值(-1)存储在内存中，0表示删除Cookie，使Cookie即时失效
- 修改Cookie只需要发送同 name 的 Cookie 覆盖即可

### 2.3 Cookie存储中文数据
- tomcat8之前不能直接存储中文数据，8.0 之后可以
- tomcat8之前可以采用URL编码来存储

### 2.4 设置Cookie的共享范围
- 默认情况下，不同的 **虚拟目录** 的web项目中Cookie是不能共享的     
- 设置共享范围：`setPath(String path)`，不设置时，**默认共享范围是当前的虚拟目录**                
- 不同tomcat服务器之间共享：`setDomain(".baidu.com")` ，所有一级域名是baidu.com 的服务器都可以共享
~~~java
    Cookie c = new Cookie("msg", "hello");
    //Cookie持久化存储到硬盘30秒
    c.setMaxAge(30);
    //设置Cookie被访问的范围：web根目录，目录下所有项目都可以访问       
    c.setPath("/");
    response.addCookie(c);
~~~

### 2.5 Cookie的特点
- 存储在浏览器端，不是很安全，能被修改      
- 浏览器对于单个Cookie 的**大小有限制(4KB)**，对同一个域名下的**总 Cookie 数量也有限制**（20个）            
- 一般用于存储少量的不太敏感的数据，**一般用作不登陆的情况下对客户端的身份识别**            



## 3. 服务端会话技术 Session
在**一次会话**的多次请求间共享数据，将数据保存在服务器端的对象中：HttpSession        
- `HttpSession request.getSession()` ： 获取Session对象
- `Object getAttribute(String name)` ： 获取Session中的共享数据       
- `void setAttribute(Stirng name, Object value)` ： 设置共享数据
- `void removeAttribute(String name)` ： 删除共享数据

### 3.1 一次会话中多次获取的Session对象是同一个
- Session的实现是依赖于Cookie的
- 第一次获取Session时Cookie没有Session信息，服务器内存中就会创建一个新的Session对象，并把Session的id存放到响应头中(set-cookie:JSESSIONID=xxxxxxx)    
- 之后客户端访问服务器时，请求头中便携带了这个Session的id

### 3.2 客户端重启后Session的变化       
- 客户端重启后，默认Session会重新获取   
- 如果要客户端重启后Session依然不变，需要手动创建一个Cookie，传入JSESSIONID，设置存活时间   

~~~java
    //获取session
    HttpSession session = request.getSession();
    //设置JSESSIONID
    Cookie c = new Cookie("JSESSIONID", session.getId());
    //设置Cookie的存活时间
    c.setMaxAge(60);
    response.addCookie(c);
~~~

### 3.3 服务器重启后Session的变化       
- 服务器重启后需要重新创建Session对象，所以地址值不会是同一个       
- **Session的钝化：** 在服务器正常关闭之前，将Session对象序列化到硬盘
- **Session的活化：** 服务器重启后，将Session文件转化为内存中的Session对象，反序列化
- tomcat**正常关闭**时，**自动钝**化到work目录的SESSIONS.ser，重新启动时**自动活化**，并删除该文件      
- IDEA关闭服务器时也会钝化，但是启动服务器时会删除work目录，重新创建work，所以不能活化      

### 3.4 Session的存活时间       
- `session.invalidate()` ： 主动调用，销毁session
- session默认的失效时间是30分钟  
- 可以在web.xml 中设置存活时间   
~~~xml
    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>
~~~

### 3.5 Session的特点       
- 用于存储一次会话的多次请求的数据，存在**服务器端**，**安全**       
- 可以存储**任意类型，任意大小**的数据。**Session存储的是Object**，**Cookie存储的是String**


## 4. JSP       
Java Server Pages：java服务器端页面             
- 既可以定义html标签，又可以定义java代码，用于简化书写
- JSP其实就是一个servlet，帮我们写好了其他的html标签

### 4.1 JSP脚本
- **<%    Java代码    %>** : 定义的java代码在service方法中，和service方法中一样使用
- **<%!   Java代码    %>** ：定义的java代码在jsp转换后的java类的成员位置，可以定义成员变量和成员方法 
- **<%=   Java代码    %>** ：定义的java代码会直接输出到页面上 out.print("Java代码")

### 4.2 JSP的内置对象
在jspu任免中不需要获取和创建，可以直接使用的对象，9个           

**域对象**          
| 变量名      | 真实类型           | 域的范围                 |
| ----------- | ------------------ | ------------------------ |
| pageContext | PageContext        | 当前页面，可以获取其他对象                 |
| request     | HttpServletRequest | 一次请求的多个资源(转发) |
| session     | HttpSession        | 一次会话的多个请求间     |
| application | ServletContext     | 所有用户间共享数据       |

**其他对象**

| 变量名    | 真是类型            | 说明|
| --------- | ------------------- | ------- |
| response  | HttpServletResponse ||
| out       | JspWriter           | 字符输出流对象，可以直接将数据输出到页面上，<br>但是response.getWriter输出永远在out前 (**tomcat服务器<br>响应客户端前先找response的缓冲区，再找out的缓冲区数据**)<br>一般jsp中都用out输出 |
| page      | Object              | 当前页面(Servlet)的对象，this  |
| config    | ServletConfig       | Servlet的配置对象        |
| exception | Throwable           | isErrorPage=true的页面才可用     |


### 4.3 指令        
- 用于配置JSP页面，导入资源文件  
- 指令的格式：`<%@ 指令名称 属性名1=属性值1 属性名2=属性值2 ... %>`

**指令的种类**              
- **page：配置JSP页面**
  - contentType ： 设置响应的contentType，包括MIME类型和编码方式        
  - language： 设置语言
  - buffer：设置缓冲区大小
  - import：导包
  - errorPage：指明错误页面地址，当前页面发生异常时自动跳转到错误页面
  - idErrorPage：指示当前页面是否是错误页面，标志了错误页面时才能用内置对象exception
- include：导入页面包含的资源文件
- **taglib：** 导入资源，一般用作**导入标签库**                 

### 4.4 EL表达式        
Expression Language 表达式语言          
可以替换和简化JSP页面中java代码的编写       
- `${表达式}`  
~~~jsp
<body>
${3>4}
\${3>4}
</body>
~~~
输出：false ${3>4}
- 如果要忽略页面中所有的EL表达式可以设置jsp的page：`isELIgnored="true"`，忽略单个直接在 $ 前加 \ 转义即可           

#### 4.4.1 EL中的运算符     
- 算数运算符：+、-、*、/(div)、%(mod)           
- 比较运算符：>、<、>=、<=、==、!=
- 逻辑运算符：&&(and)、||、!
- **空运算符符：** empty，用于判断字符串、集合、数组对象是否为**null或者长度是0**    

#### 4.4.2 EL获取值
- EL表达式只能从域对象中获取值
- `${域名称.键名}`      
- `${键名}` : 从小的域开始依次查询是否有键名
- pageScope域       --> pageContext
- requestScope域    --> request
- sessionScope域    --> session
- application域     --> application(ServletContext)

#### 4.4.3 EL获取对象的属性
- 是通过**对象的属性**来获取的(getter方法)      
- 通过定义get方法，提供给EL表达式获得相应的数据     
~~~jsp
<body>
<%
    User user = new User();
    user.setUsername("小明");
    user.setPassword("xiaoming");
    request.setAttribute("user",user);
%>

${requestScope.user}                <%--User{username='小明', password='xiaoming'}  没有toString方法则获得hash地址--%>
${requestScope.user.username}       <%--小明--%>
</body>
~~~

#### 4.4.4 EL获取List和Map中的数据      
- List：`${list[index]}`  索引越界时直接输出空字符串，不会报错          
- Map：`${map.key}` 或者 `${map["key"]}`

#### 4.4.5 EL中的隐式对象   
1. 四个域对象：pageScope、requestScope、sessionScope、application       
2. pageContext：可以获取jsp的其他8个内置对象，如 `${pageContext.request.contextPath}` 动态获取虚拟目录      

### 4.5 JSTL标签        
JavaServer Pages Tag Library ， JSP标准标签库，用于简化和替换JSP页面上的java代码                
多数时候用于展示整个list中的元素

- 需要导入jst相关jar包
- 引入标签库：taglib指令 `<%@ taglib %>`           
- 通过标签使用
#### 4.5.1 常用的JSTL标签       
- if：`<c:if test="表达式"> 表达式为真时展示的内容 </c:if>`，test接收的表达式返回**boolean**，if标签没有else   
- choose：相当于 java 中的 switch 语句，使用when标签做判断      
- foreach：相当于 java 中的 for 语句        
> 1. begin : 表示开始值(包含)
> 2. end ： 结束值(包含)
> 3. var ： 临时变量
> 4. step ： 步长
> 5. varStatus ： 循环状态对象            
>        varStatus.index ： 容器中元素的索引，从0开始        
>        varStatus.count ： 循环计次，从1开始

- choose示例：
~~~jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%--
    使用when标签做判断
--%>
<%
    request.setAttribute("number",3);
%>
123
<c:choose>
    <c:when test="${requestScope.number==1}">星期一</c:when>
    <c:when test="${requestScope.number==2}">星期二</c:when>
    <c:when test="${requestScope.number==3}">星期三</c:when>
    <c:when test="${requestScope.number==4}">星期四</c:when>
    <c:when test="${requestScope.number==5}">星期五</c:when>
</c:choose>
</body>
</html>
~~~

- if 和 forEach 示例：
~~~jsp
<%@ page import="java.util.ArrayList" %>
<%@ page import="java.util.List" %>
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<html>
<head>
    <title>Title</title>
</head>
<body>
<%
    List list = new ArrayList();
    list.add("aaa");
    list.add("bbb");
    list.add("ccc");
    request.setAttribute("list",list);
%>
<c:if test="${not empty requestScope.list}">
    <c:forEach items="${list}" var="str" varStatus="s">
        ${s.index} ${s.count} ${str} <br>
    </c:forEach>
</c:if>
<div>=============================================</div>
<c:forEach begin="1" end="10" var="i" step="3" varStatus="s">
${i}  ${s.index}  ${s.count}<br>
</c:forEach>
</body>
</html>

<%-- 
输出：
0 1 aaa
1 2 bbb
2 3 ccc
=============================================
1 1 1
4 4 2
7 7 3
10 10 4
--%>
~~~

- 使用forEach遍历列表展示在表格中   

~~~jsp
<body>
<%
    List userList = new ArrayList<>();
    userList.add(new User("小明","xiaoming"));
    userList.add(new User("小强","xiaoqiang"));
    userList.add(new User("小张","xiaozhang"));
    request.setAttribute("list",userList);
%>

<table align="center">
    <tr>
        <th>编号</th>
        <th>用户名</th>
        <th>密码</th>
    </tr>
    <c:forEach items="${requestScope.list}" var="user" varStatus="s">
        <c:if test="${s.count%2==0}">
            <tr>
                <td>${s.count}</td>
                <td>${user.username}</td>
                <td>${user.password}</td>
            </tr>
        </c:if>
        <c:if test="${s.count%2!=0}">
            <tr bgcolor="aqua">
                <td>${s.count}</td>
                <td>${user.username}</td>
                <td>${user.password}</td>
            </tr>
        </c:if>

    </c:forEach>
</table>
</body>
~~~



# 过滤器和监听器        

## 1. 过滤器Filter   
- 对浏览器的访问进行拦截，完成一些统一的特殊处理                
- 一般用作完成通用的操作，如：**登陆验证**、**统一编码处理**、**敏感字符过滤**  

### 1.1 多滤器使用步骤
- 定义类，实现 Filter 接口，覆写接口的抽象方法  
- 配置拦截路径：注解或者web.xml配置   

### 1.2 配置        

#### 1.2.1 配置拦截路径
- 通过 web.xml 配置
~~~xml
    <filter>
        <filter-name>demo1</filter-name>
        <!-- 配置对应的类为Filter -->
        <filter-class>cn.web.filter.Demo1Filter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>demo1</filter-name>
        <!-- 配置拦截路径 -->
        <url-pattern>/*</url-pattern>
    </filter-mapping>
~~~

- 通过注解配置：value是最重要的属性，就是urlPatterns，所以可以直接写 `value="/demo1"`，如果只有一个属性时value也可以省略：
~~~java
@WebFilter("/*")
public class Demo1Filter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {}

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {filterChain.doFilter(servletRequest, servletResponse);}

    @Override
    public void destroy() {}
}
~~~ 


|拦截内容|配置方式|
|---|---|
|拦截具体的资源  | /xx/xx/index.jsp  
|拦截目录        | /user/*
|拦截后缀名      |  *.jsp
|拦截所有资源    |   /*

#### 1.2.2 配置拦截方式

1. web.xml配置

2. 注解配置：设置 `dispatcherTypes` 属性的值
    - REQUEST：默认值，浏览器直接请求资源才被拦截
    - FORWARD：转发访问资源
    - INCLUDE：包含访问资源
    - ERROR：错误跳转资源
    - ASYNC：异步访问资源

~~~java
//配置浏览器直接请求资源和转发请求资源都会被拦击
@WebFilter(value="/*", dispatcherTypes = {DispatcherType.REQUEST, DispatcherType.FORWARD})
public class Demo1Filter implements Filter {...}
~~~



### 1.3 Filter执行流程      
- 执行顺序：首先执行doFilter，然后执行放行后的资源，最后再回来执行过滤器放行代码下面的代码
- doFilter 方法，在每次请求被拦截的资源时都会执行       
- init 方法，在服务器启动后，会创建Filter对象，调用init方法，只执行一次
- destroy 方法，在服务器关闭后，Filter对象被销毁，如果服务器**正常关闭**则会执行destroy方法，只执行一次         


### 1.4 过滤器链        
过滤器的先后顺序配置：
- 注解配置：按照类名额字符串比较规则比较，值小的先执行
- web.xml配置：按`<filter-mapping>`配置顺序执行

### 1.5 过滤器实现登陆验证          
- 注意要判断uri是否包含**登陆相关资源**，排除 css/js/图片/验证码等资源
~~~java
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        //request和response强转
        HttpServletRequest request = (HttpServletRequest)req;

        //获取资源请求路径
        String uri = request.getRequestURI();

        //判断uri是否包含登陆相关资源，注意排除 css/js/图片/验证码等资源
        if(uri.contains("/login.jsp")||uri.contains("/loginServlet")||uri.contains("/checkCodeServlet")
                ||uri.contains("/js/")||uri.contains("/css/")||uri.contains("/fonts")){
            chain.doFilter(req, resp);
        }else{
            //不包含则验证是否登陆
            Boolean hasLogin = (Boolean) request.getSession().getAttribute("hasLogin");
            //注意判断空指针
            if(hasLogin!=null && hasLogin==true){
                chain.doFilter(req, resp);
            }else{
                request.setAttribute("login_msg","您尚未登陆，请登陆！");
                request.getRequestDispatcher("/login.jsp").forward(request, resp);
            }
        }
    }
~~~

### 1.6 动态代理      
   
- 使用设计模式增强对象的功能：装饰模式、代理模式            

#### 1.6.1 代理模式
- 代理模式：代理对象代理真实对象，达到增强真实对象功能的目的            
- 代理的实现可以通过静态代理和动态代理
- 静态代理：有专门的类文件描述代理模式
- 动态代理：在内存中形成代理类      

#### 1.6.2 动态代理
- 代理对象和真实对象要实现相同的接口
- 通过 `Proxy.newProxyInstance()` 获得代理对象       
- 使用代理对象来调用方法    
- invoke方法的参数： proxy是代理对象， method就是代理对象调用的方法通过反射被封装为的对象， args是代理对象调用方法传入的参数列表        

~~~java
public interface Demo1 {
    String show1(int i);
    String show2(int i);
}

public class Demo1Impl implements Demo1{
    public String show1(int i){
        System.out.println("show1......"+i);
        return "show1......"+i;
    }
    public String show2(int i){
        System.out.println("show2......"+i);
        return "show2......"+i;
    }
}

public class Demo1Proxy {
    public static void main(String[] args) {
        Demo1 demo1 = new Demo1Impl();
        Demo1 proxy_demo1 = (Demo1) Proxy.newProxyInstance(demo1.getClass().getClassLoader(), demo1.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("enhanced....");
                System.out.println(method.getName());
                System.out.println(Arrays.toString(args));
                return null;
            }
        });

        String s1 = proxy_demo1.show1(2);
        System.out.println(s1);
        String s2 = proxy_demo1.show2(1);
        System.out.println(s2);
    }
}

/* 输出：
enhanced....
show1
[2]
null            //真实对象的方法并未被调用，按代理对象的return返回
enhanced....
show2
[1]
null
*/
~~~

- 使用 `method.invoke` 调用真实对象执行方法，返回真实对象的返回值

~~~java
    public static void main(String[] args) {
        Demo1 demo1 = new Demo1Impl();
        Demo1 proxy_demo1 = (Demo1) Proxy.newProxyInstance(demo1.getClass().getClassLoader(), demo1.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("enhanced....");
                Object o = method.invoke(demo1, args);
                return o;
            }
        });

        String s1 = proxy_demo1.show1(2);
        System.out.println(s1);
    }
/*
enhanced....
show1......2    //真实对象的返回值
show1......2    //打印的代理对象的返回值
*/

~~~

- **增强方式：** 增强**参数列表**、增强**返回值类型**、增强**方法体执行逻辑**           

~~~java
    public static void main(String[] args) {
        Demo1 demo1 = new Demo1Impl();
        Demo1 proxy_demo1 = (Demo1) Proxy.newProxyInstance(demo1.getClass().getClassLoader(), demo1.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                //对show1方法进行增强
                if(method.getName().equals("show1")){
                    System.out.println("show1 has been enhanced....");
                    //增强参数
                    int i = (int)args[0];
                    i +=1000;
                    String o = (String)method.invoke(demo1, args);
                    //增强返回值
                    return o+"........hahaha";
                }else{
                    //其他方法原样执行
                    return method.invoke(demo1, args);
                }

            }
        });

        String s1 = proxy_demo1.show1(2);
        System.out.println(s1);
        proxy_demo1.show2(1);
    }
/*
show1 has been enhanced....
show1......2
show1......2........hahaha
show2......1
*/
~~~

### 1.7 动态代理实现敏感词过滤

- 替换掉敏感词汇后，需要转发，request 对象没有setParameter 方法，需要对 request 对象进行增强     

~~~java
public class SensitiveWordsFilter implements Filter {
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain) throws ServletException, IOException {
        //创建代理对象增强 getParameter 方法
        ServletRequest proxy_req = (ServletRequest)Proxy.newProxyInstance(req.getClass().getClassLoader(), req.getClass().getInterfaces(), new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                if (method.getName().equals("getParameter")) {
                    //增强返回值
                    String value = (String)method.invoke(req, args);
                    if(value!=null){
                        for(String s:sensitiveWordsList){
                            if(value.contains(s)){
                                value = value.replaceAll(s, "*");
                            }
                        }
                    }
                    System.out.println(value);
                    return value;
                }else{
                    //对其他方法原样执行
                    return method.invoke(req, args);
                }
            }
        });

        chain.doFilter(proxy_req, resp);
    }

    //在init中加载敏感词汇
    private List<String> sensitiveWordsList = new ArrayList<>();   //敏感词汇集合
    public void init(FilterConfig config) throws ServletException {
        //加载并读取文件，按行添加到集合中
        String realPath = config.getServletContext().getRealPath("/WEB-INF/classes/敏感词汇.txt");

        BufferedReader reader = null;
        try {
            reader = new BufferedReader(new FileReader(realPath));

            String line = null;
            while((line = reader.readLine())!=null){
                sensitiveWordsList.add(line);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if(reader!=null) {
                try {
                    reader.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public void destroy() {
    }

}
~~~

## 2. 监听器Listener

### 2.1 事件监听机制
- 事件源：事件发生的地方
- 监听器：对象
- 注册监听：将事件、事件源、监听器绑定在一起。当事件源上发生某个事件后，执行监听器的代码        

### 2.2 监听器的配置
- web.xml 注册监听
~~~xml
<listener>
    <listener-class>cn.web.listener.ContextLoaderListener</listen-class>
</listener>
~~~

- 注解配置
~~~java
@WebListener

~~~
### 2.2 ServletContextListener  
- 监听ServletContext对象的创建和销毁
- `void contextInitialized(ServletContextEvent sce)` : ServletContext对象创建后调用该方法
- `void contextDestroyed(ServletContextEvent sce)` ： ServletContext对象被销毁后调用该方法























