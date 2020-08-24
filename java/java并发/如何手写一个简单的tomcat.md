## 如何手写一个简单的tomcat

### 一、大概的思路

- **提供Socket服务**

  Tomcat的启动，必然是Socket服务，只不过它支持HTTP协议而已！

- **进行请求的分发**

  要知道一个Tomcat可以为多个Web应用提供服务，很显然，Tomcat可以把URL下发到不同的Web应用。

- **需要把请求和响应封装成request/response**

  我们在Web应用这一层，可从来没有封装过request/response 的，我们都是直接使用的，这就是因为Tomcat已经为你做好了！

### 二、具体代码

- **封装请求对象**

```java
public class MyRequest {
    private String url;
    private String method;

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getMethod() {
        return method;
    }

    public void setMethod(String method) {
        this.method = method;
    }

    public MyRequest(InputStream inputStream) throws IOException {
        String httpRequest = "";
        byte[] httpRequestBytes = new byte[1024];
        int length = 0;
        if((length = inputStream.read(httpRequestBytes))>0){
            httpRequest = new String(httpRequestBytes,0,length);
        }

        String httpHead = httpRequest.split("\n")[0];
        url = httpHead.split("\\s")[1];
        method = httpHead.split("\\s")[0];
        //System.out.println(this);
    }
}
```

这里可以清楚的看到，通过输入流，对HTTP 协议进行解析，拿到了HTTP请求头的方法以及URL



- **封装响应对象**

```java
public class MyResponse {
    private OutputStream outputStream;

    public MyResponse(OutputStream outputStream){
        this.outputStream = outputStream;
    }
    public void write (String content) throws IOException {
        StringBuffer httpResponse = new StringBuffer();
        httpResponse.append("HTTP/1.1 200 OK\n")
                .append("Content-Type: text/html\n")
                .append("\r\n")
                .append("<html><body>")
                .append(content)
                .append("</body></html>");
        outputStream.write(httpResponse.toString().getBytes());
        outputStream.close();
    }
}
```

基于HTTP协议的格式进行输出写入。



- **Servlet 请求处理基类**

```java
public abstract class MyServlet {
    public abstract void doGet(MyRequest myRequest,MyResponse myResponse);

    public abstract void doPost(MyRequest myRequest,MyResponse myResponse);

    public  void service(MyRequest myRequest,MyResponse myResponse){
        if(myRequest.getMethod().equalsIgnoreCase("POST")){
            doPost(myRequest,myResponse);
        }else if(myRequest.getMethod().equalsIgnoreCase("GET")){
            doGet(myRequest,myResponse);
        }
    }
}
```

Servlet常见的doGet/doPost/service方法。



- **Servlet 实现类**

```java
public class FindGirlServlet extends MyServlet {
    @Override
    public void doGet(MyRequest myRequest, MyResponse myResponse) {
        try {
            myResponse.write("get girl......");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void doPost(MyRequest myRequest, MyResponse myResponse) {
        try {
            myResponse.write("post girl......");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class HelloWorldServlet extends MyServlet {
    @Override
    public void doGet(MyRequest myRequest, MyResponse myResponse) {
        try {
            myResponse.write("get world......");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void doPost(MyRequest myRequest, MyResponse myResponse) {
        try {
            myResponse.write("post world......");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

提供这2个具体的Servlet实现，只是为了后续的测试！



- **Servlet 配置**

```java
public class ServletMapping {
    private String servletName;
    private String url;
    private String clazz;

    public ServletMapping(String servletName, String url, String clazz) {
        this.servletName = servletName;
        this.url = url;
        this.clazz = clazz;
    }

    public String getServletName() {
        return servletName;
    }

    public void setServletName(String servletName) {
        this.servletName = servletName;
    }

    public String getUrl() {
        return url;
    }

    public void setUrl(String url) {
        this.url = url;
    }

    public String getClazz() {
        return clazz;
    }

    public void setClazz(String clazz) {
        this.clazz = clazz;
    }
}
```

```java
public class ServletMappingConfig {
    public static List<ServletMapping> servletMappingList = new ArrayList<>();

    static {
        servletMappingList.add(new 		ServletMapping("findGirl","/girl","TomcatTest.FindGirlServlet"));
        servletMappingList.add(new ServletMapping("helloworld","/world","TomcatTest.HelloWorldServlet"));
    }
}
```



在servlet开发中，会在web.xml中通过<servlet>和<servlet-mapping>来进行指定哪个URL交给哪个servlet进行处理



- **启动类**

```java
public class MyTomcat {
    private int port = 8080;
    private Map<String,String> urlServletMap = new HashMap<>();
    public MyTomcat(int port){
        this.port = port;
    }
    public void start(){
        //初始化 URL与对应处理的servlet的关系
        initServletMapping();
        ServerSocket serverSocket = null;
        try {
            serverSocket = new ServerSocket(port);
            System.out.println("MyTomcat is start...");

            while(true){
                Socket socket = serverSocket.accept();
                InputStream inputStream = socket.getInputStream();
                OutputStream outputStream= socket.getOutputStream();

                MyRequest myRequest = new MyRequest(inputStream);
                MyResponse myResponse = new MyResponse(outputStream);

//                请求分发
                dispath(myRequest,myResponse);

                socket.close();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(serverSocket != null){
                try {
                    serverSocket.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void initServletMapping(){
        for (ServletMapping servletMapping: ServletMappingConfig.servletMappingList) {
            urlServletMap.put(servletMapping.getUrl(),servletMapping.getClazz());
        }
    }

    private void dispath(MyRequest myRequest,MyResponse myResponse){
        String clazz = urlServletMap.get(myRequest.getUrl());
        System.out.println(clazz);
        //反射
        try {
            Class<MyServlet> myServletClass = (Class<MyServlet>) Class.forName(clazz);
            MyServlet myServlet = myServletClass.newInstance();
            myServlet.service(myRequest,myResponse);
        } catch (ClassNotFoundException | InstantiationException | IllegalAccessException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        new MyTomcat(8080).start();
    }
}
```



- 测试

浏览器输入网址：http://localhost:8080/girl

页面打印结果：get girl......

