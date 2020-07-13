# 八、JavaWeb

## 目录

  * [目录](#目录)
  * [1、文件上传](#1文件上传)
    + [保存数据到bean中](#保存数据到bean中)
      - [优化版：](#优化版)
  * [2、会话技术](#2会话技术)
  * [2.1 Cookie](#21-Cookie)
    + [2.1.1案例1：](#211案例1)
    + [2.1.2案例2：](#212案例2)
    + [2.1.3cookie的设置](#213cookie的设置)
      - [2.1.3.1 设置MaxAge](#2131-设置MaxAge)
      - [2.1.3.2 设置path](#2132-设置path)
      - [2.1.3.3 设置domain](#2133-设置domain)
    + [2.1.4 cookie细节](#214-cookie细节)
  * [2.2 Session](#22-session)
    
    + [2.2.1 Session的使用](#221-Session的使用)
    
    + [2.2.2问题一：关闭浏览器，session对象会销毁吗？](#222问题一：关闭浏览器，session对象会销毁吗？)
    
      + [2.2.3问题二：关闭浏览器，session里面的数据会怎么样？](#223问题二：关闭浏览器，session里面的数据会怎么样？)
      + [2.2.4问题三：关闭服务器，session对象会销毁吗？](#224问题三：关闭服务器，session对象会销毁吗？)
      + [2.2.5存取数据](#225存取数据)
      + [2.2.6session中的数据默认是会话级别的，关闭浏览器则失效](#226session中的数据默认是会话级别的，关闭浏览器则失效)
      + [2.2.7关闭服务器，session中的数据会丢失吗？](#227关闭服务器，session中的数据会丢失吗？)
      + [2.2.8session的生命周期](#228关闭服务器，session中的数据会丢失吗？)
      + [2.2.9案例1：购物车](#229案例1：购物车)
        - [2.2.9.1Druid.properties文件放在哪](#2291druidproperties-----)
        - [2.2.9.2Dbutils](#2292dbutils)
          * [2.2.9.2.1关于传不传datasource的讨论](#22921-----datasource---)
      + [2.2.10案例2：登录](#2210案例2：登录)
      * [3、JSP](#3-jsp)
      * [3.1jsp语法](#31jsp语法)
        + [3.1.1jsp表达式](#311jsp表达式)
        + [3.1.2jsp脚本片段](#312jsp----)
        + [3.1.3jsp声明](#313jsp--)
        + [3.1.4jsp注释](#314jsp--)
      * [3.2jsp九大对象](#32jsp九大对象)
        + [3.2.1pageContext对象](#321pagecontext--)
          - [3.2.1.1API](#3211api)
        + [3.2.2out对象](#322out--)
      * [4、Listener监听器](#4-Listener监听器)
        + [4.1如何去编写一个listener呢](#41-------listener-)
        + [4.2回调案例](#42----)
      * [5、Filter过滤器](#5-Filter过滤器)
        + [5.1filter的功能](#51filter---)
        + [5.2如何编写Filter](#52----filter)
        + [5.3Filter的生命周期](#53filter-----)
        + [5.4Servlet和filter如何关联在一起](#54servlet-filter-------)
          - [5.4.1filter和servlet设置相同url-pattern,servlet代码不执行](#541filter-servlet----url-pattern-servlet-----)
          - [5.4.2filter可以设置/*吗？](#542filter--------)
          - [5.4.3filter的调用顺序](#543filter-----)
        + [5.5整个请求的处理流程](#55---------)
      * [6、JSON](#6-json)
        + [6.1java语言如何操作json](#61java------json)
      * [7、MVC](#7-mvc)
        + [7.1注册案例](#71----)
        + [7.2MVC](#72mvc)
        + [7.3三层架构](#73----)

## 1、文件上传

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<form action="/upload" enctype="multipart/form-data" method="post">
    <input type="text" name="username"><br>
    <input type="password" name="password"><br>
    <input type="file" name="image"><br>
    <input type="submit">
</form>
</body>
</html>
```



### 保存数据到bean中

首先需要定义一个User对象

```java
public class User {

    private String username;

    private String password;

    private String image;

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public String getImage() {
        return image;
    }

    public void setImage(String image) {
        this.image = image;
    }

    @Override
    public String toString() {
        return "User{" +
                "username='" + username + '\'' +
                ", password='" + password + '\'' +
                ", image='" + image + '\'' +
                '}';
    }
}
```

#### 优化版：

html->UploadServlet

```java
@WebServlet("/upload")
public class UploadServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //request.setCharacterEncoding("utf-8");
        response.setContentType("text/html;charset=utf-8");
        //post里面写代码
        boolean result = ServletFileUpload.isMultipartContent(request);
        if (!result) {
            response.getWriter().println("不包含上传的文件");
            return;
        }
        Map map = FileUploadUtils.parseRequest(request);
        User user = new User();
        try {
            BeanUtils.populate(user, map);
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }
        //转发给另外一个servlet，将其内容显示出来
        request.setAttribute("user", user);
        request.getRequestDispatcher("/view").forward(request, response);
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

```java
public class FileUploadUtils {

    public static Map parseRequest(HttpServletRequest request){
        DiskFileItemFactory factory = new DiskFileItemFactory();
        ServletContext servletContext = request.getServletContext();
        File repository = (File) servletContext.getAttribute("javax.servlet.context.tempdir");
        factory.setRepository(repository);

        // Create a new file upload handler
        ServletFileUpload upload = new ServletFileUpload(factory);
        //如果上传的文件名中文乱码，可以这么解决
        //upload.setHeaderEncoding("utf-8");
        // bytes  1024 bytes
        //upload.setFileSizeMax(1024);
        Map map = new HashMap();
        try {
            List<FileItem> items = upload.parseRequest(request);
            //对items进行迭代
            Iterator<FileItem> iterator = items.iterator();
            while (iterator.hasNext()){
                //每一个item其实就是一个input框的封装
                FileItem item = iterator.next();
                if(item.isFormField()){
                    //当前是普通的form表单数据
                    processFormField(item, map);
                }else{
                    //当前是上传的文件
                    processUploadedFile(item, map, request);
                }
            }

        } catch (FileUploadException e) {
            e.printStackTrace();
        }
        return map;
    }

    /**
     * 处理上传的文件逻辑
     */
    private static void processUploadedFile(FileItem item, Map map, HttpServletRequest request) {
        String fieldName = item.getFieldName();
        String fileName = item.getName();
        String UPLOAD_BASE = "upload";
        fileName =  UUID.randomUUID().toString() + "-" + fileName;
        int hashCode = fileName.hashCode();
        String hexString = Integer.toHexString(hashCode);
        char[] chars = hexString.toCharArray();
        for (char aChar : chars) {
            UPLOAD_BASE = UPLOAD_BASE + "/" + aChar;
        }
        System.out.println(fieldName + " === " + fileName);
        UPLOAD_BASE = UPLOAD_BASE + "/" + fileName;
        String realPath = request.getServletContext().getRealPath(UPLOAD_BASE);
        File file = new File(realPath);
        if(!file.getParentFile().exists()){
            file.getParentFile().mkdirs();
        }
        try {
            //保存文件到硬盘中
            item.write(file);
            //保存什么路径呢？
            //img src='/应用名/资源名'
            map.put(fieldName, UPLOAD_BASE);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 处理普通的form表单数据
     * 只需要获取name和value值即可
     * 上传文件之后，不要再用getParameter去获取请求参数了
     * 有没有中文乱码问题呢？
     * @param item
     * @param map
     */
    private static void processFormField(FileItem item, Map map) {
        String name = item.getFieldName();
        String value = null;
        try {
            value = item.getString("utf-8");
            //如果传入进来的是checkbox，那么下面需要变更一下
            map.put(name, value);
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        System.out.println(name + " = " + value);
    }

}

```

分发数据

```java
@WebServlet("/view")
public class ViewServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        User user = (User) request.getAttribute("user");
        response.setContentType("text/html;charset=utf-8");
        response.getWriter().println("<div>用户名：" + user.getUsername() + "</div>");
        response.getWriter().println("<div>密码：" + user.getPassword() + "</div>");
        response.getWriter().println("<div>头像：<img src='" + request.getContextPath() + "/" + user.getImage() + "'></div>");

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

存在的问题：

* 图片显示不出来？

1. 是因为上传图片还没有写入硬盘，然后就被转发到view页面，所以显示不出来



## 2、会话技术

> 网络请求过程中，会话，客户端发起一个请求
>
> 到达服务器，接下来又发送了一个请求，又到达了服务器，服务器是区分不了客户端的，在服务器眼里，客户端全都是一样的。为什么出现这种情况？
>
> 因为**http的无状态性**
>
> 最严重的后果是购物车里面的数据有哪些、登录的是谁？
>
> 但是实际上这些情况并没有发生，原因在于存在会话技术

## 2.1 Cookie

> 一小串数据，数据由服务器生成，然后在相应的过程中传给客户端(以*set-cookie* 响应头的形式发送给客户端),客户端及时保存下来，等到下次访问服务器时(以Cookie请求头的形式发送给服务端)，就会把这个cookie给携带上。

```java
@WebServlet("/cookie")
public class ServletCookie extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //生成一个cookie
        Cookie cookie = new Cookie("name","lisi");
        //响应给客户端，以set-Cookie响应头
        //自己完全可以构建一个set-Cookie响应头，但是实际上没有必要
        //因为response已经帮助我们封装好了
        //这行代码的含义其实就是帮助我们构建set-Cookie响应头
        response.addCookie(cookie);
    }
}
```

![](img/cookie1.png)

![](img/cookie2.png)

### 2.1.1案例1：

```java
@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        String username = request.getParameter("username");
        Cookie cookie = new Cookie("username", username);
        response.addCookie(cookie);
        response.getWriter().println("登录成功，即将跳转至个人主页");
        response.setHeader("refresh","2;url=" + request.getContextPath() + "/info");
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

```java
@WebServlet("/info")
public class InfoServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //应当从请求头中拿到cookie
        //getHeader去一点一点解析，但是也没有必要
        //因为request也帮助我们封装好了
        response.setContentType("text/html;charset=utf-8");
        Cookie[] cookies = request.getCookies();
        if(cookies != null){
            for (Cookie cookie : cookies) {
                if("username".equals(cookie.getName())){
                    response.getWriter().println("欢迎您，" + cookie.getValue());
                }
            }
        }
    }
}
```

过程：

登录login.html,登录成功之后，写入cookie信息，返回给客户端保存，同时发起一个定时啊刷新，跳转至info页面，客户端就会把cookie请求头给带上，服务端取出key为username的cookie value值。

### 2.1.2案例2：

任务：打印出上次访问该页面的时间

```java
@WebServlet("/last")
public class ServletLastLogin extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Cookie[] cookies = request.getCookies();
        if(cookies != null){
            for (Cookie cookie : cookies) {
                if("lastLogin".equals(cookie.getName())){
                    String value = cookie.getValue();
                    Date date = new Date(Long.parseLong(value));
                    response.getWriter().println("last login: " + date);
                }
            }
        }
        //cookie的value值中不能有空格
        //Cookie cookie = new Cookie("lastLogin", new Date().toString());
        Cookie cookie = new Cookie("lastLogin", System.currentTimeMillis() + "");
        cookie.setMaxAge(60 * 2);
        response.addCookie(cookie);
    }
}

```

### 2.1.3cookie的设置

为什么关闭浏览器之后，cookie就消失了？

#### 2.1.3.1 设置MaxAge

默认情况下，没有设置的话，表示负数，含义表示仅存在于浏览器内存中；设置一个正数，表示可以存活多少秒。这个时候，会存在于硬盘中持续一段时间。如果设置0呢？表示的是删除cookie。

```java
@WebServlet("/del")
public class DelCookieServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Cookie[] cookies = request.getCookies();
        if(cookies != null){
            for (Cookie cookie : cookies) {
                if("lastLogin".equals(cookie.getName())){
                    cookie.setMaxAge(0);
                    String value = cookie.getValue();
                    Date date = new Date(Long.parseLong(value));
                    response.getWriter().println("last login: " + date);
                    response.addCookie(cookie);
                }
            }
        }
    }
}
```

#### 2.1.3.2 设置path

默认情况下，如果没有设置path的话，则当访问当前域名下任意资源时，均会带上该cookie，如果想仅部分路径携带cookie，则可以通过设置path来实现。

```java
@WebServlet("/path1")
public class PathServlet1 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Cookie cookie = new Cookie("name","path");
        cookie.setPath(request.getContextPath() + "/path2");
        response.addCookie(cookie);
    }
}
```

![](img/cookie3.png)

接着访问path2

![](img/cookie4.png)





```java
@WebServlet("/path2")
public class PathServlet2 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Cookie[] cookies = request.getCookies();
        if(cookies != null){
            for (Cookie cookie : cookies) {
                if("name".equals(cookie.getName())){
                    cookie.setPath(request.getContextPath() + "/path2");
                    cookie.setMaxAge(0);
                    response.addCookie(cookie);
                }
            }
        }
    }
}
```

#### 2.1.3.3 设置domain

设置域名。浏览器有一个大的原则，你不能设置和当前应用无关的域名cookie。比如当前域名是localhost,然后你设置了一个cookie，域名是www.baidu.com。这个时候浏览器是不允许的。

父子域名的规则

Zsquirrel.com	127.0.0.1		------父域名

Video.zsquirrel.com	127.0.0.1  -------一级子域名

Story.video.zsquirrel.com	127.0.0.1 ------二级子域名

 如果你在父域名下，设置了一个cookie的domain是zsquirrel.com，那么接下来，下面所有的子域名均可以共享你的这个cookie。

```java
@WebServlet("/domain")
public class DomainServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        Cookie cookie = new Cookie("key", "domain");
        //必须显式的去配置这一句话才可以
        cookie.setDomain("zsquirrel.com");
        response.addCookie(cookie);
    }
}
```

![](img/cookie5.png)



### 2.1.4 cookie细节

1. cookie的name和value值均是字符串类型
2. cookie它只能存储少量的数据，一般不超过4k
3. 默认生成的cookie是会话级别的，仅存在于浏览器内存中，如果想要持久化保存，则需要设置一个MaxAge为正数的值，表示在硬盘存活多久；设置0表示删除cookie，但是需要注意的是，如果该cookie设置了path，那么删除时，必须也设置同样的path，否则无法删除
4. 不同浏览器之间可以共享cookie吗？？？？？？？不可以。
5. 如果有一些敏感数据，或者需要存储很大量的数据，这个时候，可以怎么办呢？Session

## 2.2 Session

>服务器给每个浏览器创建了一块区域，专门用来存放数据。只要是一个浏览器的行为，均可以把这些数据存放在这个session中。浏览器和某个sessionn对象做了一个绑定。

### 2.2.1 Session的使用

在什么情况下会创建session呢？

通过request.getSession()/getSession(boolean create)这个API来创建session对象

```java
@WebServlet("/session")
public class ServletSession extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        HttpSession session = request.getSession();
        String id = session.getId();
        System.out.println(id);
    }
}
```

![](img/session1.png)

当第一次访问/session时，set-Cookie响应头

当第二次访问/session时

![](img/session2.png)

请求头中包含了Cookie请求头，此时响应头中没有再次响应set-Cookie了

原因在于cookie请求头中返回的JSESSIONID是一个有效的session id，服务器根据id找到了对应的session对象
<<<<<<< HEAD

关闭浏览器重新打开，你会发现此时又重新生成了一个新的session

原因在于：session传递给客户端时，内部设置的cookie的maxage为负数，仅在当前会话内有效，关闭浏览器，则cookie失效，cookie失效，则请求的时候不会携带JSESSIONNID，当遇到getSession这句代码时，就会触发逻辑,去判断是否有携带一个有效的JSESSIONID，如果没有，则重新创建一个新的session对象，把session的id通过cookie传回去。

### 2.2.2问题一：关闭浏览器，session对象会销毁吗？

不会。那么会一直存在吗？其实也不会。一段时间不使用的话，会被销毁。

### 2.2.3问题二：关闭浏览器，session里面的数据会怎么样？

session对象以及session里面的数据，全部处于一种不可达的状态。不可访问到。

### 2.2.4问题三：关闭服务器，session对象会销毁吗？

会。关闭服务器或者卸载应用都会销毁session对象。

### 2.2.5存取数据

setAttribute getAttribute removeAttribute

session域、context、request域

Context：当前应用内只有一个。

Session：当前应用内有多少个浏览器，就有多少个session。范围也比较广。不同的servlet之间也可以进行共享数据。

```java
@WebServlet("/session")
public class ServletSession extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {}
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        HttpSession session = request.getSession();
        System.out.println(session);
        //cookie的那么必须为JSESSIONID
        Cookie cookie = new Cookie("JSESSIONID", session.getId());
        cookie.setMaxAge(60 * 60);
        response.addCookie(cookie);
        session.setAttribute("name","session");
        String id = session.getId();
        System.out.println(id);
    }
}
```

session中存数据，在session2中取数据

```java
@WebServlet("/session2")
public class ServletSession2 extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {}
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        HttpSession session = request.getSession();
        System.out.println(session);
        System.out.println(session.getId());
        String name = (String) session.getAttribute("name");
        System.out.println(name);
    }
}
```

### 2.2.6session中的数据默认是会话级别的，关闭浏览器则失效

创建session时，发送给浏览器的cookie是会话级别的，仅存在浏览器内存中，关闭浏览器，则cookie失效。



如果想持久化保存，则需要设置maxAge：

### 2.2.7关闭服务器，session中的数据会丢失吗？

不会。但是在IDEA中你可能会看到不同的结果，但是这是由于IDEA造成的，不是session原本的性质。

如何去观察呢？需要用到一个tomcat manager管理系统

1.webapps目录下manager不要删

2.到tomcat的配置文件 tomcat-users.xml文件

<role rolename="manager-gui"/>

 <user username="tomcat" password="tomcat" roles="manager-gui"/>

配置上面两行代码。

3.重新启动IDEA的tomcat，让他读取配置文件

4.域名:端口号/manager

到管理系统中将应用卸载，然后重新启动

![](img/session3.png)

当应用被卸载或者服务器被关闭，内存中的session数据会被序列化到硬盘中，当应用重新被加载或者服务器开启，序列化的session文件会被重新读取到内存中，里面存储的sessionid会被分给一个新的session对象，同时将数据也一并给这个session对象。

### 2.2.8session的生命周期

session的创建：

Request.getSession()/getSession(boolean create)

session的销毁：

关闭服务器、应用卸载session对象会被销毁

但是session中的数据不会丢失，session可以序列化

对于session中的数据而言，数据消失的方法有如下：

1.session.removeAttribute

2.Session.invalidate()

3.一段时间不访问，session中的数据失效

![](img/session4.png)

### 2.2.9案例1：购物车

#### 2.2.9.1Druid.properties文件放在哪

![](img/session5.png)

可以放在src目录下，那么在最终部署目录中，位于classes目录下。

位于该目录下，有一个好处，就是可以利用类加载器来获取绝对路径。

c3p0-config.xml文件,放在src目录下，文件名称不要变，这个时候就可以直接得到datasource。

```java
		Connection connection = null;
        PreparedStatement psmt = null;
        ResultSet resultSet = null;
        try {
            Class.forName("com.mysql.jdbc.Driver");
             connection = DriverManager.getConnection("", "", "");
             psmt = connection.prepareStatement("select * from product");
             resultSet = psmt.executeQuery();
            List<Product> productList = new ArrayList<>();
            while (resultSet.next()){
                String name = resultSet.getString("name");
                double price = resultSet.getDouble("price");
                Product product = new Product(name, price);
                productList.add(product);
            }
            //处理完毕list拿到
        } catch (ClassNotFoundException | SQLException e) {
            e.printStackTrace();
        }finally {
            if(connection != null){
                try {
                    connection.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(psmt != null){
                try {
                    psmt.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
            if(resultSet != null){
                try {
                    resultSet.close();
                } catch (SQLException e) {
                    e.printStackTrace();
                }
            }
        }
```

上面是标准的JDBC代码。但是实际开发过程中我们不会去这么写。

频繁的创建销毁connection对象，顶不住。数据库连接池

```java
//druid.properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/22_db?characterEncoding=utf-8
username=root
password=123456
filters=stat
initialSize=2
maxActive=300
maxWait=60000
timeBetweenEvictionRunsMillis=60000
minEvictableIdleTimeMillis=300000
validationQuery=SELECT 1
testWhileIdle=true
testOnBorrow=false
testOnReturn=false
poolPreparedStatements=false
maxPoolPreparedStatementPerConnectionSize=200
```

```java
//DruidUtils
package com.cskaoyan.cart.utils;

import com.alibaba.druid.pool.DruidDataSourceFactory;

import javax.sql.DataSource;
import java.io.InputStream;
import java.sql.Connection;
import java.sql.SQLException;
import java.util.Properties;

public class DruidUtils {

    private static DataSource dataSource;

    static {
        //Druid 符合java数据库连接的规范的 所以肯定有一个datasource
        //如果放在WEB-INF下，必须要提供context或者相关
        //但是如果这么做，工具类就无法在se项目中使用
        //可以利用类加载器去获取一个绝对路径
        //类加载器有一个API可以直接定位到classes目录
        try {
            //要求就是文件必须放置在src目录下
            InputStream inputStream = DruidUtils.class.getClassLoader().
                    getResourceAsStream("druid.properties");
            Properties properties = new Properties();
            properties.load(inputStream);
            dataSource = DruidDataSourceFactory.createDataSource(properties);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static DataSource getDataSource(){
        return dataSource;
    }

    /**
     *  Class.forName("com.mysql.jdbc.Driver");
     *  connection = DriverManager.getConnection("", "", "");
     *  今后获取连接使用下面代码即可，不要自己去getConnection
     *  因为上面没有数据库连接池
     * @return
     * @throws SQLException
     */
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection();
    }
}
```

这里面我们巧妙的利用了类加载器来帮助我们找到对应的配置文件信息，类加载器加载类找到package。

#### 2.2.9.2Dbutils

##### 2.2.9.2.1关于传不传datasource的讨论

如果不传datasource，那么你需要传一个connection，那么就需要自己去手动关闭connection

如果传了datasource，那么就不需要自己去手动关闭connection

### 2.2.10案例2：登录

```java
//loginservlet
@WebServlet("/login")
public class LoginServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        String username = request.getParameter("username");
        String password = request.getParameter("password");
        if("admin".equals(username) && "admin".equals(password)){
            //登录成功，写入session域
            request.getSession().setAttribute("username", username);
            response.getWriter().println("登录成功，即将跳转");
            response.setHeader("refresh","2;url=" + request.getContextPath() + "/info");
        }
    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }
}
```

```java
@WebServlet("/info")
public class InfoServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {}

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=utf-8");
        String username = (String) request.getSession().getAttribute("username");
        response.getWriter().println("欢迎您，" + username);
    }
}
```



## 3、JSP

>jsp的特点：
>
>1.就可以把jsp当做html来对待，但是它相较于html还有很大的优势
>
>2.这里面可以嵌套java代码，然后显示动态数据
>
>3.java代码不可以随意的书写，需要在特殊的标签里才有效

## 3.1jsp语法

### 3.1.1jsp表达式

```jsp
 <%=new Date()%>
```

jsp表达式里面的java代码，会原封不动地被out.print方法包裹起来。最终翻译过后要符合java的语法。

```java
out.print(new Date());
```

### 3.1.2jsp脚本片段

```jsp
<%
//完全可以写java注释
/**
* 多行注释也ok
*/
System.out.println("脚本片段");
%>
```

```jsp
<%
for (int i = 0; i < 10; i++) {
out.print(i);
%>
<h1>hello world</h1>
<%
}
%>
```

### 3.1.3jsp声明

```jsp
<%!
//jsp声明
private String name = "zhangsan";
%>
```

### 3.1.4jsp注释

是写在主体区域里面的，不能嵌套在jsp表达式、脚本片段、声明这些里面，只能写在页面的主体部分。

![](img/jsp1.png)

html注释会原封不动的翻译到客户端。

jsp注释，在翻译称为java代码时，就会消失。

## 3.2jsp九大对象

在service方法内部会创建出来9个对象，可以直接供我们来使用。默认情况下只能看到8个，如果想看到第9个，需要设置isErrorPage=true

```jsp
<%@ page isErrorPage="true" contentType="text/html;charset=UTF-8" language="java" %>
```

>Request
>
>Response
>
>pageContext
>
>Session
>
>Exception
>
>Application--servletContext
>
>Config
>
>Out
>
>Page

### 3.2.1pageContext对象

> 九大对象中最为核心的一个对象。通过该对象可以获得其他八个对象。

#### 3.2.1.1API

本身也是一个域。Request、session、application。

该域大小是当前页面内

```JSP
<%
//通过该对象可以获得其他八个对象
pageContext.getRequest();
pageContext.setAttribute("name", "page");
%>
<%
String name = (String)pageContext.getAttribute("name")
System.out.println(name);
%>
```

2.可以给其他域赋值

````jsp
pageContext.setAttribute("name", "page", PageContext.PAGE_SCOPE);
pageContext.setAttribute("name", "request", PageContext.REQUEST_SCOPE);
pageContext.setAttribute("name", "session", PageContext.SESSION_SCOPE);
pageContext.setAttribute("name", "application", PageContext.APPLICATION_SCOPE);
````

```jsp
<%=pageContext.getAttribute("name")%>
<%=request.getAttribute("name")%>
<%=session.getAttribute("name")%>
<%=application.getAttribute("name")%>--%>
```

```jsp
request.getRequestDispatcher("/page.jsp").forward(request, response);
```

1.首先，在当前页面不用转发，直接打印

```jsp
<%=pageContext.findAttribute("name")%>
```

显示的是page

2.调用转发，然后在被转发页面打印该语句，显示的是request

3.直接访问被转发页面page.jsp，打印的是session

4.在page.jsp中调用session.invalidate，打印的是application

得出结论：

一级一级去查找。从最小域pageContext域开始，依次request、session、application域去查找，在某个域找到数据则结束，找不到则接着下一个

### 3.2.2out对象

输出数据到客户端。带有缓存功能的Writer。

冲洗条件：

1.buffer缓冲区满

2.关闭buffer缓冲区

3.页面需要响应

## 4、Listener监听器

Web：

监听对象：ServletContext

监听事件：context创建和销毁

监听器：自己编写的一个监听器

触发事件：监听器里面的代码执行

### 4.1如何去编写一个listener呢

1.编写一个类实现ServletContextListener接口

2.注册该Listener

```java
@WebListener
public class MyServletContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("context init");
        ServletContext servletContext = servletContextEvent.getServletContext();
        //context域
        //读取配置文件等操作，将参数放入context域中
        //比如配置了xxx.properties
    }
    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("context destroy");

    }
}

```

### 4.2回调案例

员工老板。员工工作工作完成之后汇报给老板。

```java
public interface Leader {

    void report();
}
```

```java
public class Boss implements Leader {

    public void report(){
        System.out.println("工作完成，向老板汇报");
    }
}

```

```java
public class Manager implements Leader {
    @Override
    public void report() {
        System.out.println("工作完成，向总经理汇报");
    }
}
```

```java
public class Employee {

    private Leader leader;

    public Employee(Leader leader) {
        this.leader = leader;
    }

    public void work(){
        System.out.println("员工在工作");
        leader.report();
    }
}
```

测试

```java
public class Test {

    public static void main(String[] args) {
        //Leader leader = new Boss();
        Leader leader = new Manager();
        // listener注册然后将其注入到context中
        Employee employee = new Employee(leader);
        // context.init()---内部 listener.contextInitialized
        employee.work();
    }
}
```

![](img/listener1.png)

## 5、Filter过滤器

### 5.1filter的功能

1.可以设置拦截或者放行(验证，是否登录，info页面仅登录可用)

2.可以在请求到达servlet之前修改request对象，也可以在响应之后修改response对象（字符编码格式）

### 5.2如何编写Filter

1.编写一个类实现javax.servlet.Filter接口

2.注册该filter(先在web.xml中根据提示完成)



### 5.3Filter的生命周期

1.Init：随着应用的启动而实例化

2.doFilter：每访问一次filter，就会执行该方法一次

3.Destroy：应用的卸载或者服务器关闭

### 5.4Servlet和filter如何关联在一起

最简单的方式就是通过url-pattern。servlet和filter设置相同的url-pattern。

为什么没有报错？

不是说url-pattern不可以设置相同的嘛？为什么filter和servlet设置同一个url-pattern不会报错？

不可以设置相同的url-pattern是因为针对的是servlet。servlet从功能上来说，是开发动态web资源，做出响应的，如果多个servlet配置了相同url-pattern，究竟应该选择哪个来执行呢？

但是filter功能上来说，和servlet完全不同，filter定位拦截、过滤，而不是做出响应。

更多的是功能上的一个差异。功能上的一个定义。

```
<filter>
<filter-name>firstFilter</filter-name>
<filter-class>FilterDemo</filter-class>
</filter>
<filter-mapping>
<filter-name>firstFilter</filter-name>
<url-pattern>/*</url-pattern>
</filter-mapping>
```

#### 5.4.1filter和servlet设置相同url-pattern,servlet代码不执行

为什么呢？

filter默认执行的是拦截操作，如果想要放行代码往下执行，必须要有这句话

```java
@Override
public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
	servletResponse.setContentType("text/html;charset=utf-8");
	System.out.println("生命周期doFilter");
	//这行代码对于filter放行代码来说至关重要
	filterChain.doFilter(servletRequest, servletResponse);
}
```

此时filter和servlet的代码均会被执行到

例子：

```java
//@WebFilter
public class FilterDemo implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("生命周期init");
    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletResponse.setContentType("text/html;charset=utf-8");
        System.out.println("生命周期doFilter");
        //这行代码对于filter放行代码来说至关重要
        filterChain.doFilter(servletRequest, servletResponse);
    }

    @Override
    public void destroy() {
        System.out.println("生命周期destroy");

    }
}
```

#### 5.4.2filter可以设置/*吗？

可以的。不会存在servlet的那些诸多烦恼。但是filter一般不会设置/。

#### 5.4.3filter的调用顺序

1.对于在web.xml中声明的filter，满足如下规律：

首先看能够处理当前请求的filter有哪些，其次再看

mapping声明的先后顺序，就是最终的调用顺序。

2.对于注解的方式，顺序是类名首字母的ASCII先后顺序。

AFilter  SFilter，先A后S

### 5.5整个请求的处理流程

比如一个叫做ROOT的应用，里面配置servlet，url-pattern叫做/servlet，同时配置了两个filter，一个AFilter叫做/*，一个BFilter叫做/servlet。都是采用注解的方式。

当访问如下请求时

http://localhost:8080/servlet

执行流程如下：

1.浏览器地址栏输入如下地址，构建一个请求报文

2.请求报文传输到指定机器的指定端口8080端口

3.被connector接收到，然后将其转成request对象，同时生成一个response对象

4.这两个对象被传给engine，engine挑选host来处理该请求

5.将这两个对象传给选好的host，host选择合适的Context来处理

6.这两个对象传给Context，请求的资源/servlet，Context根据请求的资源在当前应用下寻找合适的组件来处理该请求

7.首先先查找filter，哪些filter可以处理(匹配)该请求，将这两个filter按照一定的顺序组成一个链表

8.接下来去查找servlet，匹配到对应的servlet，然后将servlet添加到filter的链后面

9.接下来依次调用链上的组件，然后将request和response依次穿进去，先执行filter接下来执行servlet，然后执行完毕，返回

10.connector读取reponse里面的数据生成响应报文

## 6、JSON

需要记住一点：{}表示的是一个对象，[]表示的是一个数组或者集合。

### 6.1java语言如何操作json

前后端分离的概念。页面和数据分别来自于两个不同的系统。

比如一个项目：页面内来自于localhost:8080,数据来自于localhost:8084，

java语言提供对应页面所需要的数据。

数据返回就是以json字符串的形式来返回

{"name":"zhangsan", "age": 24}

可以用一个网站来校验besjon.com

专门用来校验json字符串。

```java
public static void main(String[] args) {
Person person = new Person();
person.setName("zhangsan");
person.setAge(24);
//需要把person对象转成json字符串
//var person = {"name": "zhangsan", "age": 24}
String perStr = "{\"name\":\"" + person.getName() + "\", \"age\": " + person.getAge() + "}";
System.out.println(perStr);
}
```

使用工具类来完成这些操作。

Gson google json解析工具 

Fastjson alibaba

Jackson SpringMVC框架内置

不管怎么样，完成的功能都是一样的。

下面利用Gson:

```java
public static void main(String[] args) {
Person person = new Person();
person.setName("zhangsan");
person.setAge(24);
Person person2 = new Person();
person2.setName("lisi");
person2.setAge(22);
List<Person> people = new ArrayList<>();
people.add(person);
people.add(person2);
//将java对象转成json字符串
Gson gson = new Gson();
String s = gson.toJson(person);
System.out.println(s);
String s1 = gson.toJson(people);
System.out.println(s1);

//{"name":"zhangsan","age":24}
//[{"name":"zhangsan","age":24},{"name":"lisi","age":22}]
}
```

其他操作：json数组转化为json对象

```java
public static void main(String[] args) {
//{"name":"zhangsan","age":24}
//[{"name":"zhangsan","age":24},{"name":"lisi","age":22}]
String person = "{\"name\":\"zhangsan\",\"age\":24}";
String people = "[{\"name\":\"zhangsan\",\"age\":24},{\"name\":\"lisi\",\"age\":22}]";
//将json字符串转成对应的java类型
Gson gson = new Gson();
Person person1 = gson.fromJson(person, Person.class);
System.out.println(person1);
//如何将一个数组对象的数据转成List对象的java数据类型
//1.
JsonElement jsonElement = new JsonParser().parse(people);
//根据json字符串最外侧是[]还是{}灵活的去选择getAsJsonArray还是getAsJsonObject
JsonArray jsonArray = jsonElement.getAsJsonArray();
List<Person> personList = new ArrayList<>();
for (JsonElement element : jsonArray) {
Person p = gson.fromJson(element, Person.class);
personList.add(p);
System.out.println(p);
}
//personlist --- JDBC
}
```

## 7、MVC

### 7.1注册案例

```json
public class StringUtils {
    public static boolean isEmpty(String s) {
        if(s == null || "".equals(s.trim())){
            return true;
        }
        return false;
    }
}
```

再写一个User类对象(此处省略)

再写一个filter过滤

```json
@WebServlet("/register")
public class RegisterServlet extends HttpServlet {
protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
//注册，注册的信息保存到json文件中
//取出参数，判断用户名是否一致
String username = request.getParameter("username");
String password = request.getParameter("password");
String confirmPassword = request.getParameter("confirmPassword");
//校验 是否为空 密码确认密码是否一致
if(StringUtils.isEmpty(username) || StringUtils.isEmpty(password)||
StringUtils.isEmpty(confirmPassword)){
response.getWriter().println("参数不能为空");
return;
}
if(!password.equals(confirmPassword)){
response.getWriter().println("两次密码不一致，请确认");
return;
}
User user = new User();
user.setUsername(username);
user.setPassword(password);
//封装对象
InputStream inputStream = RegisterServlet.class.getClassLoader().getResourceAsStream("user.json");
//形成一个字符串就行了
ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
byte[] bytes = new byte[1024];
int length = 0;
while ((length = inputStream.read(bytes))!= -1){
byteArrayOutputStream.write(bytes, 0, length);
}
String jsonStr = byteArrayOutputStream.toString("utf-8");
Gson gson = new Gson();
List<User> userList = new ArrayList<>();
if(!"".equals(jsonStr)){
JsonElement jsonElement = new JsonParser().parse(jsonStr);
JsonArray jsonArray = jsonElement.getAsJsonArray();
for (JsonElement element : jsonArray) {
User u = gson.fromJson(element, User.class);
//判断是否存在当前用户名
if(u.getUsername().equals(username)){
response.getWriter().println("当前用户名已经存在");
return;
}
userList.add(u);
}
}
userList.add(user);

//直接写回数据到json文件中
String information = gson.toJson(userList);
//把这个字符串写回user.json文件中
String file = RegisterServlet.class.getClassLoader().getResource("user.json").getFile();
FileOutputStream fileOutputStream = new FileOutputStream(file);
fileOutputStream.write(information.getBytes("utf-8"));
response.getWriter().println("注册成功，即将跳转至登录页");
//json文件中是否存在这个用户名
//没有用户名才注册
}

protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

}
```

//先把以前的写入到一个输出流中，转化为字符串，放入List，转化为对象，看看新的和以前的重复不重复，如果不重复那就再写入

### 7.2MVC

>数据模型（Model）：封装的是数据模型和所有基于对这些数据的操作。在一个组件中，Model往往表示组件的状态和操作状态的方法。
>
>视图（View）：封装的是对数据Model的一种显示。一个模型可以由多个视图，而一个视图理论上也可以同不同的模型关联起来。
>
>控制器（Control）：封装的是外界作用于模型的操作,  通常，这些操作会转发到模型上，并调用模型中相应的一个或者多个方法。一般Controller在Model和View之间起到了沟通的作用，处理用户在View上的输入，并转发给Model。这样Model和View两者之间可以做到松散耦合，甚至可以彼此不知道对方，而由Controller连接起这两个部分。



user对象属于model，对user对象的所有操作：比如注册、判断用户名是否存在、登录，这些都是对于user对象的操作。也算model。

View：对数据模型的显示。用户个人页面：显示的就是user的信息。Jsp。

Controller：中转站。中间人。请求到来以后，控制器做一些处理，比如校验，之后调用model的一个或者多个方法，接下来model返回结果，根据结果的不同，调用不同的视图。model和view之间就不会产生任何的影响。

### 7.3三层架构

接下来，需要将MVC进一步优化，这里面其实还有很大可优化的空间？如果方法很多，需要将每个方法里面的之前的model全部换成新的model，更改的地方还是很多。

仔细地去体会一下，为什么仅需要去修改类名，而不需要去修改方法名称以及返回值，因为我们是刻意这么做的，因为这样做的话修改的地方最少。潜意识里去做一个标准。

为何不用天生就适合做标准的呢？？？？？？？？？接口。

使用接口，不管是Json还是Mysql，其实都是在操作数据，又可以给它们重新取一个名字，叫做DAO，Data Access Object。

三层架构，究竟是哪三层呢？

说法很多，展示层、业务层service、数据层dao

 Controller、service、dao

有一个疑惑：

感觉有controller和dao已经可以实现功能了，为什么还需要有service呢？

对于一些逻辑非常简单的模块，没有service感觉没有任何影响，但是

如果对于一些业务逻辑比较复杂的模块，service就体现出它的功能了。

比如分页查询数据结果。

分页显示图中数据。

一共多少条记录，每页多少条，一共多少页，同时展示当前页应该显示的数据条目。

