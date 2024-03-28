# SpringMVC响应数据

## 一、handler方法分析

**待补充**...

## 二、页面跳转控制

**待补充**...

### 2.1 快速返回模板视图

1. **开发模式回顾**

    ![3-SpringMVC响应数据0](image/3-SpringMVC%E5%93%8D%E5%BA%94%E6%95%B0%E6%8D%AE/1711441511731.png)  

2. **jsp技术了解**

    ![3-SpringMVC响应数据1](image/3-SpringMVC%E5%93%8D%E5%BA%94%E6%95%B0%E6%8D%AE/1711441566625.png)  

### 2.2 转发和重定向

**待补充**...

## 三、返回JSON数据【*重点】

### 3.1 前置准备

导入jsckson依赖

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.16.0</version>
</dependency>
```

添加json数据转换器 @**EnableWebMvc**

```java
@EnableWebMvc
@Configuration
@ComponentScan("org.alan.jsp")
public class MvcConfig implements WebMvcConfigurer {
    //@EnableWebMvc自动配置handlerMapping handlerAdapter json转换器
}
```

### 3.2 @Responsebody

* **@Responsebody** 数据直接放入响应体放回，也不会走视图解析器
* 快速查找视图，转发和重定向都不生效

```java
@Controller
@RequestMapping("json")
@ResponseBody//方法返回值转化为json格式
public class JsonController {

    //对象 -> json -> {}
    //集合 -> json -> []
    @GetMapping("data")
    public User data(){
        User user = new User();
        user.setName("Alan");
        user.setAge(22);
        return user; //user -> handlerAdapter -> json -> @ResponseBody -> json返回
    }

    //json返回集合
    @GetMapping("data1")
    public List<User> data2(){
        User user = new User();
        user.setName("Alan");
        user.setAge(22);

        User user1 = new User();
        user1.setName("Wang");
        user1.setAge(22);

        List<User> users = new ArrayList<>();
        users.add(user);
        users.add(user1);
        return users;
    }
}
```

### 3.3 @RustController

* **@RustController** == @Controller + @@Responsebody

```java
//@Controller
@RequestMapping("json")
//@ResponseBody
@RustController
public class JsonController {

}
```

## 四、返回静态资源处理

1. Config配置，开启静态资源查找

   访问请求发出后，DispathcherServlet先在HnadlerMapping中查找资源，没有再向DefaultServletHandler查找，通过handleRequest(路径)查找并转发项目内部静态资源

    ```java
    @EnableWebMvc
    @Configuration
    @ComponentScans({@ComponentScan("org.alan.jsp"),@ComponentScan("org.alan.json")})
    public class MvcConfig implements WebMvcConfigurer {

        //@EnableWebMvc自动配置handlerMapping handlerAdapter json转换器

        //视图解析器，指定前后缀
        /*@Override
        public void configureViewResolvers(ViewResolverRegistry registry)*/

        //开启静态资源查找
        //
        @Override
        public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
            configurer.enable();
        }
    }
    ```
