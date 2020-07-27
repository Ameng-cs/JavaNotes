# SpringMVC

### @RquestMapping的属性

- **value:**告诉SpringMVC这个方法用来处理的是哪一个请求

  - 标注在类上:为该类所有的方法的地址都加了一个前缀地址指定基准路径,用前缀增加请求名称的重用性
  - 标注在方法上:指定处理请求的方法
  - 其中引号中的/可以省略,都表示从当项目下开始(习惯加上)

- **method:**限定请求方法(GET/POST),不是规定的请求方式就会报错 ,默认是所有请求都可以

- **params:**确定请求参数

  - params={"参数名"}:发送请求时必须带一个指定名称的参数   
  - params={"!参数名"}:发送请求时必须不带指定名称的参数   
  - params={"key=value/key=!value"}:发送请求时必须带一个指定了值的键值对   
  - params={"key=value","参数名"}:发送请求时,请求参数必须满足多规则

- **Header:**规定请求头,其规则跟params相同

- **consumes:**指定接收内容类型是哪种请求:规定请求头中的Content-type

- **produces:**告诉浏览器返回的内容类型是什么:给响应头中加上Content-type

- **@PathVariable**    

  - 带占位符的url    路径上可以有占位符,语法就是在任意路径的地方写一个{变量名}

    ```java
    @RequestMapping("/user/{变量名}")
    public String pathVariableTest(@PathVatiable("变量名")String x){
        System.out.println("路径上的占位符为:"+x);
        return "success";
    }  
    ```

    

### 获取请求中的各种信息

- **@RequestParams**

  - 获取请求中参数的值
  - @RequestParam("变量名")String name ==-->== name = request.getParameter("变量名")
  - @RequestParam(value="key",required=true,defalutVlue="val")
    - value:指定要获取的请求参数的key        
    - required:(true/false)指定这个参数是否是必须的        
    - defaultValue:规定默认值,不规定的话默认为NULL

  - 与**@PathVariable**的区别:
    - @PathVariable是获取路径上的值        
    - @RequestParams是获取?后的参数的值

- **@RequestHeader**

  - 获取请求头中的某个key的值
  - @RequestHerder("key")String head ==-->== head = request.getHeader("key")

- **@CookieValue**

  - 获取某个cookie的值
  - @CookieValue("JSESSIONID")String jid  ==-->==   Cookie[] cookies = request.getCookies()循环遍历cookies找到需要的cookies

```java
@RequestMapping("/user")
public String Test(@RequestParam("变量名")String name,
                   @RequestHerder("User-Agent")String head
                   @CookieValue("JSESSIONID")String jid){
    System.out.println("获取到的参数值:"+name);
    System.out.println("获取到的头信息的User-Agent的值为:"+head);
    System.out.println("获取到的cookie的JSESSIONID为:"+jid);
    return "success";
}   
```

- **如果我们的请求参数是一个POJO,SpringMVC会自动为POJO赋值**

```java
@RequestMapping("/book")
public String Test(Book book){
    //Spring会帮我们将book对象封装好
    System.out.println("我要保存的图书:"+book);
    return "success";
}   
```

- 在方法参数上直接写下原始API可以直接调用以下对象
  - HttpServletRequest
  - HttpServletResponse
  - HttpSession,Locale
  - InputStream    
  - OutputStream
  - Reader  
  - Writer

```java
@RequestMapping("/book")
public String Test(HttpServletRequest request){
    System.out.println("我获取到了原生的API:"+request);
    request.setAttribute("key","value");
    return "success";
}  
```



### 向页面传输数据

- **利用Map(接口),Model(接口),ModelMap(类)**

  - Map,Model,ModelMap最终都是==BindingAwareModelMap==在工作,相当于BindingAwareModelMap 保存的东西都会被保存在==request域==中

    ```java
    @RequestMapping("handle01")
    public String handle01(Map<String,Object> map) {
        map.put("msg","你好");
        return "succeed";
    }
    @RequestMapping("handle02")
    public String handle02(Model model) {
        model.addAttribute("msg","你好呀");
        return "succeed";
    }
    @RequestMapping("handle03")
    public String handle03(ModelMap modelMap) {
        modelMap.addAttribute("msg","你好压抑");
        return "succeed";
    }
    ```

- 将返回值编程ModelAndView类型

  - 既包含==视图信息==(页面地址)也包含==模型的数据(==给页面带的数据) ,数据会存在==request域中== 

    ```java
        @RequestMapping("handle04")
        public ModelAndView handle04() {
            //之前发的返回值我们就叫视图名,视图名解析器会帮我们拼串得到真实地址
           ModelAndView modelAndView = new ModelAndView("succeed");
           modelAndView.addObject("msg","你好呀");
           return modelAndView;
        }  
    ```



### 视图解析

在设置了路径拼接的情况下,如果想要返回上级目录的文件怎么办

- 使用相对路径:       

  ```java
  return "../../hello"    
  ```

- 使用forward(转发)/redirect(重定向):+绝对路径,

  - forward后接的路径并不会进行拼接,如果不加/就是相对路径,容易出问题        

  ```
  return "forward:/hello.jsp" 
  ```

- 使用forward(转发)/redirect(重定向):+另外一个方法的mapping

  - 表示把这个请求交给mapping为handle02的方法处理        

  ```
  return "forward:/handle02"   
  ```

###  

### 拦截器

SpringMVC提供了拦截器:允许在运行目标方法之前进行一些拦截,或者在目标方法进行之后 进行一些处理==(类似于Filter)== 

- HandlerInterceptor(接口)    

  - ==boolean preHandle():==在目标方法运行之前调用;返回为true:放行;返回为false:不放行    
  - ==postHandle():== 在目标方法运行之后调用;目标方法调用之后    
  - ==afterCompletion():== 在请求玩成之后;来到目标页面之后;真正的放行;资源响应之后

- 在dispatchServlet中配置拦截器

  ```xml
  <mvc:interceptors>
      <!--配置拦截器,默认拦截所有请求-->
      <mvc:interceptor>
          <!--只拦截test请求-->
          <mvc:mapping path="/test"/>
          <bean class="com.spring.test.MyInterceptor"></bean>
      </mvc:interceptor>
  </mvc:interceptors> 
  ```

- 拦截器的正常运行流程:preHandler-->目标方法-->postHandler-->目标页面-->afterCompletion 其他流程:

- 只要其中某一位置出错或是没放行,后面的流程除了afterCompletion其他的都不会执行.

- 多拦截器顺序:先进的后出

- 什么时候用Filter什么时候用拦截器       

  - 如果某些功能需要其他组件配合完成,我们就使用拦截器        
  - 其他情况可以使用Filter,Fliter可有在非Spring环境下运行   

### AJAX

- 依赖:
  - jackson-annotations-2.1.5.jar    
  - jackson-core-2.1.5.jar    
  - jackson-databind-2.1.5.jar

- 配置

  - 在处理AJAX的方法上添加==@ResponseBody==:
    - SpringMVC会将返回的数据放到响应体中,如果是对象会自动将对象转为JOSN格式             
    - 若返回值改为ResponseEntity<>,则可以返回一个自定义的响应

  - 在参数上标注==@RequestBody==:
    - 获取一个请求的请求体
    - 若把参数类型改为HttpEntity str,则str不仅包括了请求体信息,还包括了请求头信息 

  - 在bean的日期属性上标注==@JsonFormat(pattern="yyyy-MM-dd")==:
    - 让SpringMVC在封装JSON的时候会按照该Format显示日期
  - 在bean的对应属性上标注==@JsonIgnore==:
    - 如果该属性是对象的话,SpringMVC 就不会再现身该属性了



### 文件上传

- 配置

```xml
<!--配置文件上传解析器 id必须是multipartResolver-->
<bean class="org.springframework.web.multipart.commons.CommonsMultipartResolver" id="multipartResolver">
    <!-- 设置上传大小限制-->
    <property name="maxUploadSize" value="#{1024*1024*20}"></property>
    <!-- 设置默认编码  -->
    <property name="defaultEncoding" value="utf-8"></property> 
</bean>
```

- 要上传的表单类型改为

```
enctype="multipart/form-data"
```

- 单文件上传

```java
@RequestMapping(value = "/upload")
public String upload(@RequestParam(value = "user",required = false)String user,:用来接收普通参数
                     @RequestParam(value = "heading",required = false)MultipartFile file, :用来接收文件参数
                     Model model:用来存储数据)  {
    System.out.println("普通表单项:"+user);
    System.out.println("表单上文件项的name值:"+file.getName());
    System.out.println("上传的文件的大小:"+file.getSize());
    System.out.println("文件名"+file.getOriginalFilename());
    System.out.println("文件的类型"+file.getContentType());

    try {
        file.transferTo(new File("/"+file.getOriginalFilename()));
        model.addAttribute("msg","文件上传成功了!");
    } catch (IOException e) {
        e.printStackTrace();
        model.addAttribute("msg","文件上传失败了!");
    }
    return "forward:/index.jsp";
}  
```

- 多文件上传

```java
@RequestMapping(value = "/upload")
public String upload(@RequestParam(value = "user",required = false)String user,:用来接收普通参数
                     @RequestParam(value = "headimg",required = false)MultipartFile[] files, :用来接收文件参数
                     Model model:用来存储数据)  {
    System.out.println("普通表单项:"+user);
    for(MultipartFile file:files){
        if(!file.isEmpty()){
            System.out.println("表单上文件项的name值:"+file.getName());
            System.out.println("上传的文件的大小:"+file.getSize());
            System.out.println("文件名"+file.getOriginalFilename());
            System.out.println("文件的类型"+file.getContentType());
            try {
                file.transferTo(new File("/"+file.getOriginalFilename()));
                model.addAttribute("msg","文件上传成功了!");
            } catch (IOException e) {
                e.printStackTrace();
                model.addAttribute("msg","文件上传失败了!");
            }
        }
    }
    return "forward:/index.jsp";
}   
```

### 数据绑定

==<mvc:annotation-driven />== 该注释的功能:

- 会自动注册RequestMappingHandlerMapping、RequestMappingHandlerAdapter  、ExceptionHandlerExceptionResolver 支持使用了像`@RquestMapping`、`ExceptionHandler`等等的注解的controller 方法去处理请求
- 提供以下功能的支持
  - 支持使用 ConversionService 实例对表单参数进行类型转换    
  - 支持使用 @NumberFormat annotation、@DateTimeFormat 注解完成数据类型的格式化    
  - 支持使用 @Valid 注解对 JavaBean 实例进行 JSR 303 验证   
  - 支持使用 @RequestBody 和 @ResponseBody 注解
- <mvc:annotation-driven />与\<mvc:default-servlet-handler/>
  - 两个都没加:动态资源(@RequestMapping映射的资源)能访问;静态资源(.html,.js,.img...)不能访问 
  - 加了\<mvc:default-servlet-handler/>,没加<mvc:annotation-driven />:则静态资源可以访问,动态资源不能访问 
  - <mvc:annotation-driven />与\<mvc:default-servlet-handler/>都加了:动态和静态资源都能访问

### 视图解析器的配置

```xml
<!--视图解析器配置-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/pages/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
```



### Web.xml的相关配置

```xml
<!--这个标签用来指定另外的IOC容器applicationContext.xml-->
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/applicationContext.xml</param-value>
</context-param>
<listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<!--配置dispatcher前端控制器-->
<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--Servlet启动加载:servlet原本是第一次访问创建对象
    load-on-startup:服务器启动的时候创建对象,值越小,优先级越高,越先创建对象-->
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>dispatcherServlet</servlet-name>
    <!--改为/*和/都表示拦截所有的请求;/会拦截所有请求除了jsp
    /*的范围更大:还会拦截到*.jsp这些请求,一旦拦截jsp页面就不能显示了
    /会拦截所有请求除了jsp-->
    <url-pattern>/</url-pattern>
</servlet-mapping>
<!--该过滤器用来解决乱码问题,一般配在请他filter之前-->
<filter>
    <filter-name>CharacterEncoding</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!--encoding:解决POST请求乱码-->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!--forceEncoding:解决响应乱码-->
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncoding</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<!--该过滤器用来支持REST风格的URL-->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```

- **DispatcherServlet的url-pattern的配置:**
  - /   : 拦截所有请求,不拦截jsp页面(*.jsp请求)    
  - /\* : 拦截所有请求,拦截jsp页面(*.jsp请求)
  - 为什么配/会拦截所有除了.jsp请求呢?
    - 处理\*.jsp时候Tomcat做的事:(所有项目的小Web.xml都是继承于大Web.xml的),DefaultServlet是Tomcat中处理静态资源的(除了jsp和servlet)服务器的大Web.xml中有一个DefaultServlet的url-pattern=/我们配置中的前端控制器url-pattern=/这两个url-pattern相同相当于我们前端控制器的url-pattern=/禁用了服务器中DefaultServlet的url-pattern=/ 静态资源就会来到DispatcherServlet(前端控制器)寻找哪个方法的RequestMapping与请求相同,所以DispatcherServlet就会拦截所有的请求但是大Web.xml中还有一个JSPServlet他的url-pattern指定了这个Servlet是来处理*.jsp的,而没有被DispatcherServlet重写所以这就解释了为什么只有jsp不会被拦截

