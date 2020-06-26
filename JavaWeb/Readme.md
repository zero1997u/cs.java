- [JavaWeb](#javaweb)
  * [1.文件上传](#1----)
    + [保存数据到bean中](#-----bean-)
      - [优化版：](#----)

# JavaWeb

## 1.文件上传

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









