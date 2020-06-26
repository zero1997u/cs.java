# 八、JavaWeb

## 目录

- [八、JavaWeb](#八-javaweb)
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
